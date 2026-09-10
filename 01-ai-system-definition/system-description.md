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
  directly and are not shown their score. `TODO` decide whether patients are informed that
  such a system is in use (transparency obligation depends on classification).

## 4. Inputs

| Category | Examples | Source |
|---|---|---|
| Utilisation | ED visits, admissions, no-shows over trailing 12 months | Partner-clinic EHR extract |
| Lab-value trends | HbA1c, eGFR, BP direction of travel | Partner-clinic EHR extract |
| Condition flags | Diabetes, CHF, COPD, CKD (coded) | Partner-clinic EHR extract |
| Demographic buckets | Age band, sex; `TODO` decide whether any proxy for race, language, or deprivation is included, and document the bias rationale either way | Partner-clinic EHR extract |

All fields are de-identified at the partner clinic before transfer (same pipeline and
controls as the population-health dataset in the Cloud Risk Assessment lab). No free text,
no direct identifiers.

## 5. Outputs

- `risk_score` (integer 0 to 100)
- `risk_tier` (low / moderate / high), thresholds `TODO` after calibration
- `top_features` (the 3 to 5 features that contributed most, for clinician context)
- Delivery: dashboard ranked list, refreshed `TODO` (nightly batch assumed)

## 6. Model and method

- Approach: supervised binary classification (event in next 90 days: yes / no), gradient
  boosted trees (SageMaker built-in XGBoost).
- Training / validation / test split: `TODO`
- Performance: `TODO` after build. Record AUROC, AUPRC, calibration, and **subgroup
  performance** across the demographic buckets.
- Retraining cadence: `TODO`

## 7. Human oversight

- The clinician sees the score, the tier, the top contributing features, and the underlying
  data. `TODO` confirm the dashboard shows enough for the clinician to sense-check a score.
- The clinician can ignore or override the priority order with no friction.
- `TODO` define what stops a clinician from over-relying on the score (automation bias):
  training, a visible "recommendation only" label, periodic review of outreach outcomes.

## 8. Deployment architecture

See [`data-flow-diagram.md`](data-flow-diagram.md). Summary: nightly de-identified extract
lands in an S3 bucket (KMS encrypted) -> a scoring job calls the SageMaker Serverless
Inference endpoint -> scores written back to the dashboard datastore -> surfaced to
clinicians. All components tagged `project=meridian-ai`.

## 9. Data governance

- Provenance: synthetic dataset modelled on partner-clinic EHR extracts; de-identified by
  construction.
- Representativeness: `TODO` document which populations the training data does and does not
  represent, and the limitation that follows.
- Retention: `TODO`
- Bias assessment: `TODO` this is a control in its own right (NIST AI RMF Measure; EU AI Act
  Art. 10). Plan the subgroup analysis before the build so the data is captured.

## 10. Lifecycle stage and change management

- Current stage: development / pre-production.
- `TODO` define the gate to production, the approval owner, and how model or data changes are
  versioned and reviewed.

## 11. Third-party and foundation components

- SageMaker built-in XGBoost algorithm container (AWS-provided).
- `TODO` list any other libraries or pretrained artefacts and their licences.

## 12. Logging and record-keeping

- CloudTrail records API calls (endpoint invoke, model and data access).
- `TODO` decide what per-decision record is kept (input feature vector, score, model
  version, timestamp) and for how long. The EU AI Act high-risk regime requires automatic
  logging over the system's lifetime.

## 13. Known limitations

- Trained on synthetic data; not validated on any real population.
- Predicts association, not causation; a high score is not a clinical judgement.
- `TODO` add limitations found during the build (feature gaps, calibration drift, subgroup
  gaps).
