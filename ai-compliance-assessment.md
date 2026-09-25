# AI Compliance Assessment: Meridian Rising Risk Score

**Assessor:** Callum McRae (GRC) &nbsp;|&nbsp; **Date:** 2026-09-25 &nbsp;|&nbsp; **System:** Meridian Rising Risk Score v0.1

The headline report, written for a non-technical executive audience. The detail behind each
section is in the numbered folders and in Eramba.

## 1. Executive summary

The Meridian Rising Risk Score (MRRS) is a machine-learning model that scores each patient
in a partner clinic's panel for near-term health risk, so clinicians can prioritise
proactive outreach. It is assessed here as a **high-risk AI system under the EU AI Act**,
and separately against the **NIST AI RMF**, with an optional ISO/IEC 42001 overlay planned
once the primary frameworks are substantially met.

The system is pre-production. A minimal real version was built on AWS (SageMaker) to test
this assessment against actual technical evidence rather than design intent alone, trained,
evaluated, and torn down. That build **confirmed, rather than merely projected, one of the
Critical risks**: the model measurably understates risk for a disadvantaged patient subgroup,
and separately surfaced a new finding (model overfitting and poor calibration) that was not
apparent before real evaluation. Of the 18 controls in scope, none are yet fully implemented,
7 are partially underway (3 by design or documentation, 4 because a real evaluation has run
and found gaps still to fix), and 11 have not started. Ten AI-specific risks have been
identified and scored, two Critical (confirmed model unfairness to an under-served subgroup,
and clinician over-reliance on the tool's output). **Recommendation: do not deploy MRRS to a
real clinic yet.** A phased treatment plan exists to close the gaps; the two Critical risks
and the outstanding medical-device qualification question should be resolved before any
production decision is revisited.

## 2. System and scope

MRRS predicts, for each patient, the probability of a significant care event in the next 90
days, delivered as a score and tier via a nightly batch to the clinician dashboard. A
clinician decides whether to act; MRRS contacts no one and changes no record. It is trained
on the de-identified dataset from the Cloud Risk Assessment lab, intended to run on AWS
(SageMaker), and used by clinicians and care-management staff at partner clinics. Full
detail, including the governance decisions on patient notice, excluded demographic proxies,
oversight design, retention, and the production gate, is in
[`01-ai-system-definition/system-description.md`](01-ai-system-definition/system-description.md).

## 3. EU AI Act classification

MRRS is treated as **high-risk**. This is not a clean textual fit under Annex III point 5 as
currently drafted (it is not emergency triage, not a public-authority eligibility decision,
not insurance pricing), but a purposive reading, consistent with the European Commission's
draft classification guidance, brings a patient-prioritisation tool affecting access to
essential healthcare into scope, and once inside Annex III the "profiling" carve-out removes
any derogation regardless of the human-in-the-loop design. A second, independent route to
high-risk exists if MRRS qualifies as a medical device under the MDR, which is a genuinely
open question referred to regulatory counsel. Full reasoning, including the weaker contrary
reading the assessment does not rely on, is in
[`02-eu-ai-act-classification/classification.md`](02-eu-ai-act-classification/classification.md).

High-risk obligations for Annex III systems were deferred by the Digital AI Omnibus from
2 August 2026 to **2 December 2027**, deferred, not cancelled, and a reason to build the
control set ahead of the deadline rather than wait.

## 4. Control assessment

18 controls are in scope: 12 from the EU AI Act (Arts. 9-17, 26, 43, 47, 49, 72), 5 from the
NIST AI RMF (Govern, Map, Measure x2, Manage), and 1 ISO/IEC 42001 overlay check. All 18 are
built and tracked live in **Eramba** (Control Catalogue, linked to the AI risk register and
to compliance packages for each framework). Current state: **0 compliant, 7 partially
implemented, 11 not started, 0 not applicable.** Three of the partial controls (AIA-03
Technical documentation, AIA-06 Human oversight, RMF-MP-1 Context and categorisation) have a
design or documentation artefact in place that has not yet been formally reviewed. The other
four moved to Partial only after the AWS proof-of-concept build produced real evidence:
**AIA-02 / RMF-MS-2** (Data governance, Bias assessment) ran a genuine subgroup fairness
check and found a real disparity; **AIA-07 / RMF-MS-1** (Accuracy/robustness, Performance
metrics) captured real AUROC/AUPRC/calibration numbers and found the model overfits. Full
detail in [`03-control-catalogue/control-catalogue.csv`](03-control-catalogue/control-catalogue.csv)
and [`05-assessment-and-gaps/assessment.md`](05-assessment-and-gaps/assessment.md).

## 5. AI risk register

Ten AI-specific risks have been identified and scored (Likelihood x Impact, 1-5 each), nine
from the design-stage assessment plus one (AR-10) added after the AWS build surfaced it:

| Risk | Band | Summary |
|---|---|---|
| AR-01 | **Critical (16)** | Model systematically under-scores an under-served subgroup, reducing their proactive outreach. **Confirmed by the build:** absolute risk understated by roughly 9x the gap seen elsewhere for this subgroup, despite balanced ranking and recall. |
| AR-03 | **Critical (16)** | Automation bias: clinicians defer to the ranked list, patients the model does not flag get neglected |
| AR-10 | High (15) | **New finding from the build:** the model overfits (train AUC ~1.0 vs test AUC 0.75) and is poorly calibrated, degrading score reliability for every patient, not just a subgroup |
| AR-02, AR-05, AR-06, AR-08 | High (12) | Unreliable scores from stale data; undetected model drift; exposure of patient feature data or scores; an override path that is not validated under real workload |
| AR-04, AR-07, AR-09 | Medium (6-9) | Insufficient explainability to sense-check a score; patients not yet told an AI system influences their care; a now-identified third-party model component pending formal licence review |

Every risk carries a residual target set in Eramba's Treatment tab, none go to Low; a
fairness or oversight risk in a clinical ML system is never treated as fully eliminated.
Full register: [`04-ai-risk-register/ai-risk-register.csv`](04-ai-risk-register/ai-risk-register.csv).
Full evidence behind AR-01 and AR-10: [`07-aws-model/model-notes.md`](07-aws-model/model-notes.md).

## 6. Treatment plan

The 18 control gaps are grouped into **11 remediation actions**, several controls close
together under one fix, phased 0-30 / 30-90 / 90+ days:

- **0-30 days:** formalise the risk-management process (G-01); mitigate the now-confirmed
  subgroup disparity (G-02, priority raised from 30-90 days once the build confirmed it);
  formally sign off technical documentation (G-03); deploy the patient notice and deployer
  instructions (G-05).
- **30-90 days:** logging and post-market monitoring (G-04); human-oversight validation with
  clinician users (G-06); fix the overfitting and calibration gap, plus robustness and
  security testing (G-07).
- **90+ days:** quality management system (G-08); MDR qualification and conformity
  assessment (G-09); registration and declaration of conformity (G-10, blocked on G-09);
  the ISO/IEC 42001 overlay (G-11, deferred by design).

Full plan with owners and residual risk:
[`06-treatment-plan/treatment-plan.csv`](06-treatment-plan/treatment-plan.csv).

## 7. Residual risk and recommendation

**Do not proceed to production yet.** The two Critical risks (AR-01 fairness, now confirmed
by measurement rather than projected; AR-03 automation bias) both trace to controls with no
current implementation (G-01, G-02, G-05, G-06), and the medical-device qualification
question (G-09) is unresolved, which leaves the correct conformity route undetermined. A
third risk, AR-10 (overfitting and calibration), was not on the original register and only
surfaced once a real model was built and evaluated, itself a reason to keep building and
testing before any wider claim of readiness. None of this blocks continued development.

**Conditions for revisiting the production decision:**
1. G-01, G-02, G-05 and G-06 implemented and their linked risks (AR-01, AR-03, AR-07, AR-08)
   re-scored at or below their residual targets.
2. A formal MDR qualification opinion obtained (G-09), even if the conformity assessment
   itself is still in progress.
3. Technical documentation and the system context review formally signed off (G-03).

Assuming steady progress, re-assessment is scheduled to align with the asset's own review
date, **2026-12-11**, at the production-gate milestone.

## 8. Limitations of this assessment

- **All data is synthetic**, including the held-out disadvantage variable used for the
  subgroup fairness check. The *pattern* it revealed (balanced ranking, disparate absolute
  calibration) is a real and general failure mode; the specific numbers are not evidence
  about any real population.
- **The AWS build was minimal and torn down**, one training run, one small evaluation
  endpoint, 2,400 synthetic rows. It is real technical evidence, not a design assumption, but
  it is a proof of concept, not a production-grade evaluation (no cross-validation, no
  hyperparameter search, no adversarial testing yet, see gap G-07).
- **Tooling limited two build choices**, documented as trade-offs rather than hidden: the
  SageMaker execution role used the broad `AmazonSageMakerFullAccess` managed policy rather
  than a hand-scoped one, and the endpoint was deployed as a small real-time instance rather
  than Serverless Inference because the local AWS CLI predated serverless support. Neither
  reflects the intended production architecture. Full detail:
  [`07-aws-model/model-notes.md`](07-aws-model/model-notes.md).
- **Single assessor.** The classification, risk scoring, and control assessment are all my
  own judgement; a real engagement would have a second reviewer, especially for the MDR
  question.
- **Regulatory instability.** The Commission's Art. 6(5) classification guidelines are still
  in draft, and the high-risk compliance deadline has already moved once (Digital AI
  Omnibus). The classification here is the best current reading, not a settled one.
- **Single-user Eramba instance.** GRC Contact and Control Operator Contact are the same
  account for every record, a real deployment would separate oversight from execution.
