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
| Partially implemented (efficacy 40%) | 3 |
| Not started (efficacy 0%) | 15 |
| Not applicable | 0 |

MRRS is pre-production. No control is fully implemented yet, which is the honest and
expected state at this stage, not a finding in itself. The three partially-implemented
controls are the ones where a design or documentation artefact already exists even though
the operating control does not yet run:

| Control | Framework | What exists | What is missing |
|---|---|---|---|
| AIA-03 Technical documentation | EU AI Act Art. 11 + Annex IV | `system-description.md` covers most required content | Not finalised or formally reviewed and approved |
| AIA-06 Human oversight | EU AI Act Art. 14 | Oversight design specified (labelling, feature display, dismiss/reorder) | Not built or validated with clinician users |
| RMF-MP-1 Context and categorisation | NIST AI RMF MAP 1-5 | System description and classification cover context, purpose, affected parties | Not formally reviewed or approved |

## Gaps

Every one of the 18 controls is currently a gap. They are grouped into 11 remediation
actions in [`../06-treatment-plan/treatment-plan.csv`](../06-treatment-plan/treatment-plan.csv),
since several controls close together under one fix.

| Gap | Controls | Priority window |
|---|---|---|
| G-01 Formalise the risk-management process | AIA-01, RMF-GV-1, RMF-MG-1 | 0-30 days |
| G-02 Build and validate bias/data-quality controls | AIA-02, RMF-MS-2 | 30-90 days |
| G-03 Finalise technical documentation and context review | AIA-03, RMF-MP-1 | 0-30 days |
| G-04 Build logging and post-market monitoring | AIA-04, AIA-09 | 30-90 days |
| G-05 Deploy patient transparency and deployer instructions | AIA-05, AIA-12 | 0-30 days |
| G-06 Validate human oversight with clinician users | AIA-06 | 30-90 days |
| G-07 Run performance, robustness and security testing | AIA-07, RMF-MS-1 | 30-90 days |
| G-08 Stand up a quality management system | AIA-08 | 90+ days |
| G-09 Resolve MDR qualification, run conformity assessment | AIA-10 | 90+ days |
| G-10 Register and declare conformity | AIA-11 | 90+ days, blocked on G-09 |
| G-11 ISO/IEC 42001 overlay gap check | ISO-42001-A | 90+ days, deferred by design |

Full descriptions, recommended actions, owners and residual risk are in the treatment plan.
