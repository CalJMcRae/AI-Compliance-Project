# AI System Description: Meridian Rising Risk Score

Written to double as the basis for the EU AI Act technical documentation (Annex IV) and the
NIST AI RMF **Map** function. `TODO` marks a decision or a value to complete, several of them
after the model is built in [`../07-aws-model/`](../07-aws-model/).

---

## 1. System identification

| Field | Value |
|---|---|
| System name | Meridian Rising Risk Score (MRRS) |
| Version | 0.1 (pre-production) |
| Provider (EU AI Act sense) | Meridian Health Analytics |
| Deployer(s) | Partner clinics that embed the Meridian dashboard |
| Intended purpose | Prioritise clinician outreach by estimating near-term ("rising") health risk for each patient in a partner clinic's panel |
| Not intended for | Diagnosis, treatment selection, automated denial or approval of care, eligibility for insurance or benefits |

## 2. Intended purpose and context of use

MRRS produces, for each patient in a partner clinic panel, a score from 0 to 100 and a tier
(low / moderate / high) representing the estimated probability of a significant care event in
the next 90 days. The output appears in the clinician dashboard as a ranked outreach list.

A clinician or care-management team member reviews the list and decides whether to act
(outreach call, care-management enrolment, appointment). **The model recommends a priority
order; a human makes every care decision.** MRRS does not contact patients, schedule
anything, or change a record on its own.

## 3. Users and affected persons

- **Users:** clinicians and care-management staff at partner clinics.
- **Affected persons:** patients in partner-clinic panels, whose likelihood of proactive
  outreach is influenced by the score. Affected persons do not interact with the system
  directly and are not shown their score.
- **Patient notice:** patients **are** informed. The partner clinic's layered privacy notice
  states that automated analysis of health data helps the care team prioritise proactive
  outreach, that it makes no care decision, and how to ask questions or object. This
  satisfies GDPR Arts. 13 to 14 even though AI Act Art. 50 likely does not compel it. Tracked
  as control AIA-05 and risk AR-07.

## 4. Inputs

| Category | Examples | Source |
|---|---|---|
| Utilisation | ED visits, admissions, no-shows over trailing 12 months | Partner-clinic EHR extract |
| Lab-value trends | HbA1c, eGFR, BP direction of travel | Partner-clinic EHR extract |
| Condition flags | Diabetes, CHF, COPD, CKD (coded) | Partner-clinic EHR extract |
| Demographic buckets | Age band and sex only | Partner-clinic EHR extract |

**No proxies for race, ethnicity, language or area deprivation are used as model features.**
Age band and sex are kept because they are standard, clinically grounded risk factors.
Including deprivation or race proxies would risk encoding existing care-access disparities
(risk AR-01) for modest predictive lift over utilisation and lab trends. The sensitive
attributes are held **separately, outside the model inputs**, and used only to measure
subgroup performance during the bias assessment (NIST AI RMF MEASURE 2.11). "Do not train on
it, but do measure outcomes across it."

All fields are de-identified at the partner clinic before transfer (same pipeline and
controls as the population-health dataset in the Cloud Risk Assessment lab). No free text,
no direct identifiers.

## 5. Outputs

- `risk_score` (integer 0 to 100)
- `risk_tier` (low / moderate / high), thresholds `TODO` set at calibration during the build
- `top_features` (the 3 to 5 features that contributed most, for clinician context)
- Delivery: dashboard ranked list, refreshed **nightly** (batch job)

## 6. Model and method

- Approach: supervised binary classification (event in next 90 days: yes / no), gradient
  boosted trees (SageMaker built-in XGBoost).
- Training / validation / test split: `TODO` set during the build (time-based split intended)
- Performance: `TODO` after build. Record AUROC, AUPRC, calibration, and **subgroup
  performance** across age, sex and the held-out sensitive attributes.
- Retraining cadence: **scheduled every 6 months**, plus an **out-of-cycle retrain** if
  monitored AUROC falls below, or subgroup disparity rises above, a threshold set at
  calibration. Feeds Art. 72 post-market monitoring (AIA-09).

## 7. Human oversight

Design measures for effective oversight (Art. 14) and against automation bias:

- The dashboard labels the output **"decision support, not a clinical judgement."**
- Every row shows the score, the tier, the top contributing features, and a link to the
  underlying data, so a clinician can sense-check any score before acting.
- A clinician can dismiss a row or reorder the list with one click and no justification
  required.
- Meridian supplies deployer training material covering the tool's purpose, limitations and
  known failure modes (AIA-05, AIA-12).
- Meridian and each clinic review a sample of outreach outcomes **quarterly**, specifically
  checking for patients who were not surfaced by the tool but subsequently deteriorated
  (guards against clinicians ignoring unlisted patients). Feeds AR-03 and AR-08.

## 8. Deployment architecture

See [`data-flow-diagram.md`](data-flow-diagram.md). Summary: nightly de-identified extract
lands in an S3 bucket (KMS encrypted) -> a scoring job calls the SageMaker Serverless
Inference endpoint -> scores written back to the dashboard datastore -> surfaced to
clinicians. All components tagged `project=meridian-ai`.

## 9. Data governance

- Provenance: synthetic dataset modelled on partner-clinic EHR extracts; de-identified by
  construction.
- Representativeness: the training population models a specific **adult primary-care** case
  mix. It does **not** represent paediatric, obstetric, or oncology-heavy panels. MRRS must
  not be deployed to a panel materially different from the training population without
  revalidation. Recorded as a limitation (s13) and a deployer instruction (AIA-05).
- Retention: feature extracts are retained **90 days rolling** then deleted. Per-decision
  records are retained per s12.
- Bias assessment: a subgroup performance analysis across age, sex and the held-out
  sensitive attributes is run before the production gate and at every retrain. It is a
  control in its own right (NIST AI RMF MEASURE 2.11; EU AI Act Art. 10) and a risk (AR-01).
  The data needed for it is captured during the build.

## 10. Lifecycle stage and change management

- Current stage: development / pre-production.
- **Production gate:** a go / no-go review chaired by the Meridian AI governance owner,
  passed only when: the classification is signed off; Arts. 9 to 15 controls are at least
  "Partial" with a plan to "Implemented"; the bias assessment shows no unmitigated material
  disparity; the human-oversight design has been validated with clinician users; and a
  deployer agreement covering AIA-12 responsibilities is signed.
- **Change management:** the model carries a semantic version and each training run records
  the dataset hash. Model or data changes are re-reviewed by the AI governance owner;
  material changes trigger a re-assessment against this document.

## 11. Third-party and foundation components

- SageMaker built-in XGBoost algorithm container (AWS-provided).
- `TODO` complete from the build: exact `xgboost`, `scikit-learn`, `pandas`, `numpy`
  versions and their licences. No pretrained weights are used. An inventory is maintained as
  part of the QMS (AIA-08) and supply-chain risk (AR-09).

## 12. Logging and record-keeping

- CloudTrail records API calls (endpoint invoke, model and data access).
- **Per-decision record:** for each scored patient-day, the record holds an input reference
  (a hash or key, not the raw vector), the score, the tier, the model version, the
  timestamp, and whether a clinician acted. Retained for the **deployer clinic's
  medical-record retention period**, and never less than the lifetime of the system, to meet
  Art. 12.

## 13. Known limitations

- Trained on synthetic data; not validated on any real population.
- Predicts association, not causation; a high score is not a clinical judgement.
- Valid only for an adult primary-care panel resembling the training population (s9).
- Sensitive attributes are deliberately excluded as inputs, so the model cannot correct for
  a disparity it cannot see; the quarterly subgroup review is the compensating control.
- `TODO` add limitations found during the build (feature gaps, calibration drift, subgroup
  gaps).
