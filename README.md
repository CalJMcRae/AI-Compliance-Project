# AI Compliance Assessment (Eramba)

A compliance assessment of a simulated clinical AI system, run end to end inside a real GRC
platform (Eramba Community). The system is assessed against the **EU AI Act (2024)** and the
**NIST AI RMF 1.0**, with an optional **ISO/IEC 42001:2023** gap overlay.

**Author:** Callum McRae &nbsp;|&nbsp; **Scenario:** Meridian Health Analytics (simulated)

> **Status: complete.** Classification, risk register (10 risks), control catalogue
> (18 controls), gap analysis and a phased treatment plan (11 remediation actions) are built
> and tracked live in Eramba. The minimal real AWS model
> ([`07-aws-model/`](07-aws-model/)) has been built, evaluated and torn down, and its
> evidence **confirmed** one of the Critical risks and surfaced a new one (model overfitting
> and poor calibration). See [`ai-compliance-assessment.md`](ai-compliance-assessment.md) for
> the full report. **Recommendation: do not deploy to production yet.**

---

## The scenario

Meridian Health Analytics builds a model, the **Meridian Rising Risk Score**, that scores each
patient in a partner clinic's panel for near-term ("rising") health risk and returns a
0 to 100 score plus a tier (low / moderate / high). The scores surface in the clinician
dashboard as a prioritised outreach list. A clinician decides whether to act; the model
recommends, the human decides.

- **Training data:** the de-identified longitudinal clinical dataset from the
  [Cloud Risk Assessment lab](https://github.com/CalJMcRae/GRC-AWS-Project) (utilisation,
  lab-value trends, chronic-condition flags, demographic buckets). Synthetic throughout.
- **Hosting:** AWS. A small tabular model (SageMaker built-in XGBoost) deployed to Serverless
  Inference, called from a thin scoring service. Everything prefixed `meridian-ai-`.
- **Affected persons:** patients in partner-clinic panels, whose care prioritisation is
  influenced by the score.
- **Users:** clinicians and care-management staff at partner clinics.

## Why this project

AI governance is a fast-growing compliance area that few entry-level candidates can
demonstrate. Doing the assessment inside a recognised GRC platform, rather than a single
document, is a second differentiator. The hard part is genuine: the EU AI Act classification
of a clinical decision-support tool is contested (Annex III high-risk versus the medical
device route under the MDR), and this project has to reason through that, not skip it.

## Repo navigation

| Folder | Contents |
|---|---|
| [`01-ai-system-definition/`](01-ai-system-definition/) | System description, data-flow diagram, lifecycle and roles |
| [`02-eu-ai-act-classification/`](02-eu-ai-act-classification/) | EU AI Act risk classification and the reasoning, including the ambiguity |
| [`03-control-catalogue/`](03-control-catalogue/) | Combined control set: EU AI Act obligations + NIST AI RMF subcategories |
| [`04-ai-risk-register/`](04-ai-risk-register/) | AI-specific risk register with likelihood / impact scoring |
| [`05-assessment-and-gaps/`](05-assessment-and-gaps/) | Control status, gaps and owners (mirrors the Eramba assessment) |
| [`06-treatment-plan/`](06-treatment-plan/) | Remediation plan per gap, with residual risk |
| [`07-aws-model/`](07-aws-model/) | The minimal real model: build notes, evidence, teardown |
| [`docs/`](docs/) | Eramba exports, diagrams, screenshots |
| `ai-compliance-assessment.md` | The headline narrative report (classification, gap analysis, treatment, residual risk) |

## Tools and frameworks

EU AI Act (2024) &middot; NIST AI RMF 1.0 &middot; ISO/IEC 42001:2023 (overlay) &middot;
Eramba Community Edition (open-source GRC platform) &middot; AWS (SageMaker Serverless
Inference, CloudTrail, KMS, IAM)

## Notes

- All data is synthetic. No real patients, clinics or organisations.
- The AWS footprint is torn down after evidence is captured; cost is controlled with a
  budget alert and serverless (scale-to-zero) inference.
