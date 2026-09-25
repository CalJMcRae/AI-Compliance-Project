# Assessment and Gap Analysis

Mirrors the assessment run in Eramba (Compliance Management > Compliance Analysis, packages
`EU AI Act (2024)`, `NIST AI RMF 1.0`, `ISO/IEC 42001 (Overlay)`). For each of the 18 controls
in [`../03-control-catalogue/control-catalogue.csv`](../03-control-catalogue/control-catalogue.csv):
status, efficacy, and the gap that feeds the treatment plan.

## Summary

| | Count |
|---|---|
| Controls in scope | 18 |
| Compliant | 0 |
| Partially implemented | 7 |
| Not started | 11 |
| Not applicable | 0 |

MRRS is pre-production. No control is fully implemented yet, which is the honest and
expected state at this stage, not a finding in itself. Seven controls are partially
implemented, three where a design or documentation artefact exists but has not been
operationalised, and four (updated after the AWS proof-of-concept build) where a real
evaluation has been run and found genuine gaps still to be fixed:

| Control | Framework | What exists | What is missing |
|---|---|---|---|
| AIA-03 Technical documentation | EU AI Act Art. 11 + Annex IV | `system-description.md` covers all required content | Not formally reviewed and approved |
| AIA-06 Human oversight | EU AI Act Art. 14 | Oversight design specified (labelling, feature display, dismiss/reorder) | Not built or validated with clinician users |
| RMF-MP-1 Context and categorisation | NIST AI RMF MAP 1-5 | System description and classification cover context, purpose, affected parties | Not formally reviewed or approved |
| AIA-02 Data and data governance | EU AI Act Art. 10 | Subgroup bias assessment run on the proof-of-concept model | Found a real disparity (AR-01); not yet mitigated |
| RMF-MS-2 Bias and fairness assessment | NIST AI RMF MEASURE 2.11 | Same assessment as AIA-02 | Same gap; mitigation is treatment gap G-02 |
| AIA-07 Accuracy, robustness and cybersecurity | EU AI Act Art. 15 | AUROC/AUPRC/calibration metrics captured | Overfitting and calibration gaps found (AR-10); adversarial/security testing not yet run |
| RMF-MS-1 Performance and trustworthiness metrics | NIST AI RMF MEASURE 2 | Same metrics as AIA-07 | Explainability display not yet built |

The two new findings (AIA-02/RMF-MS-2's subgroup disparity, and AIA-07/RMF-MS-1's
overfitting and calibration gap) came directly out of the AWS proof-of-concept build in
[`../07-aws-model/`](../07-aws-model/), evaluating a real trained model rather than a
design intent turned up problems a paper assessment could not have found.

## Gaps

Every one of the 18 controls is currently a gap. They are grouped into 11 remediation
actions in [`../06-treatment-plan/treatment-plan.csv`](../06-treatment-plan/treatment-plan.csv),
since several controls close together under one fix.

| Gap | Controls | Priority window |
|---|---|---|
| G-01 Formalise the risk-management process | AIA-01, RMF-GV-1, RMF-MG-1 | 0-30 days |
| G-02 Mitigate the confirmed subgroup disparity + data-quality checks | AIA-02, RMF-MS-2 | 0-30 days (raised from 30-90, finding confirmed) |
| G-03 Formally review and sign off technical documentation | AIA-03, RMF-MP-1 | 0-30 days |
| G-04 Build logging and post-market monitoring | AIA-04, AIA-09 | 30-90 days |
| G-05 Deploy patient transparency and deployer instructions | AIA-05, AIA-12 | 0-30 days |
| G-06 Validate human oversight with clinician users | AIA-06 | 30-90 days |
| G-07 Run performance, robustness and security testing | AIA-07, RMF-MS-1 | 30-90 days |
| G-08 Stand up a quality management system | AIA-08 | 90+ days |
| G-09 Resolve MDR qualification, run conformity assessment | AIA-10 | 90+ days |
| G-10 Register and declare conformity | AIA-11 | 90+ days, blocked on G-09 |
| G-11 ISO/IEC 42001 overlay gap check | ISO-42001-A | 90+ days, deferred by design |

Full descriptions, recommended actions, owners and residual risk are in the treatment plan.
