# Minimal AWS Model: Build Notes and Evidence

The smallest real footprint that produces genuine technical-control evidence. Built,
evidence captured, torn down. Complete 2026-09-25.

## Plan

- [x] Synthetic training dataset (reuse the Cloud Risk Assessment lab data pattern)
- [x] S3 bucket `meridian-ai-data-<suffix>`, KMS encrypted, Block Public Access on
- [x] Execution role `meridian-ai-sagemaker-role` created
- [x] SageMaker built-in XGBoost training job
- [x] Deploy to a live inference endpoint (real-time, not serverless, see trade-off below)
- [x] Scoped execution role: read/write the one bucket via an inline policy, alongside the
      broader `AmazonSageMakerFullAccess` managed policy (documented trade-off below)
- [x] Confirm CloudTrail is recording (shared with the Cloud Risk Assessment account trail)
- [x] Run the full held-out test set through the live endpoint; save input and output
- [x] Capture evidence (CLI output, computed metrics, raw predictions) into `evidence/`
- [x] Record metrics: AUROC, AUPRC, calibration, subgroup performance
- [x] Tear down: delete endpoint, endpoint config, model; empty and delete the bucket;
      schedule the KMS key for deletion; delete the execution role

## Build log

- **Dataset:** synthetic, 2,400 rows (1,600 train / 400 validation / 400 test), 12 features
  (utilisation, lab-value trends, condition flags, age band, sex), no demographic proxies
  per system description decision B. One additional variable, a socioeconomic/access
  disadvantage flag, was generated to genuinely elevate a subgroup's true risk (42.3% true
  event rate vs 24.9% for everyone else in the test set) but deliberately **excluded** as a
  model input, so it could be used to run a real subgroup fairness check (AR-01, RMF-MS-2).
- **S3 bucket:** `meridian-ai-data-328064416121`, KMS-encrypted (`alias/meridian-ai`, key
  rotation enabled), Block Public Access on all four settings, tagged `project=meridian-ai`.
- **Execution role:** `meridian-ai-sagemaker-role`, trust limited to `sagemaker.amazonaws.com`.
  Attached `AmazonSageMakerFullAccess` (documented trade-off below) plus an inline policy
  scoping S3 access to only the `meridian-ai-data-*` bucket.
- **Scoping trade-off recorded:** the SageMaker execution role used the AWS-managed
  `AmazonSageMakerFullAccess` policy rather than a hand-scoped one. Hand-scoping SageMaker's
  own training/hosting permissions reliably is a real time sink, and this was a role only the
  SageMaker service could assume, for a build torn down immediately after evidence capture.
  The compensating control was the inline S3 policy, which did scope data access to the
  single bucket regardless of what the managed policy would otherwise allow.
- **KMS key policy note:** IAM roles must exist before they can be added as principals in a
  KMS key policy, so the key was policy-updated in two passes (root and `Callum-v2` first,
  the execution role added once it existed). A second gap surfaced during the first training
  attempt: `kms:Encrypt`/`Decrypt`/`GenerateDataKey`/`DescribeKey` alone were not enough,
  `kms:CreateGrant` (scoped with the `kms:GrantIsForAWSResource` condition) was also needed
  for SageMaker to attach the encrypted training volume.

### Encountered constraints, resolved

1. **SageMaker training instance quota was 0 by default** on every instance type tried
   (`ml.m5.large`, `ml.m4.xlarge`, `ml.m5.xlarge`, `ml.c5.xlarge`), an AWS anti-abuse default
   on lightly-used accounts, not a permissions error. Resolved by requesting a Service Quotas
   increase (approved same day, account-level value raised to 15).
2. **KMS `CreateGrant` was missing** on the first training attempt (`Access denied to KMS
   Key`), resolved as above.
3. **Wrong built-in-algorithm image URI** on the first successful-permissions attempt
   (`manifest unknown`): the older `632365934929...xgboost:1.7-1` path is only valid for the
   legacy 0.72-era container. The current XGBoost built-in algorithm uses a different
   registry account and repository name per region; for us-west-1:
   `746614075791.dkr.ecr.us-west-1.amazonaws.com/sagemaker-xgboost:1.5-1`.
4. **The local AWS CLI (2.0.30, from 2020) predates Serverless Inference**, so
   `create-endpoint-config` rejects `ServerlessConfig` regardless of input format. Rather than
   upgrade the CLI mid-build, the endpoint was deployed as a small **real-time** instance
   (`ml.t2.medium`) and deleted immediately after evidence capture. This is a tooling
   limitation, not an architectural decision, a production deployment would use serverless
   or auto-scaling real-time inference, documented here as the trade-off it is.

## Evidence captured

All raw files in [`evidence/`](evidence/): `train.csv`, `validation.csv`, `test.csv`,
`test_meta.csv` (held-out subgroup labels), `test-predictions-joined.csv` (predictions
joined to ground truth), `calibration-deciles.csv`, `metrics-summary.txt`,
`describe-training-job.json`, `describe-endpoint.json`.

**Training:** job `meridian-ai-xgb-20260925-094813`, `ml.m5.large`, 124 seconds.
`train:auc = 0.9999...`, `validation:auc = 0.7198`. The gap between them is overfitting
(150 rounds, max depth 5, on only 1,600 rows) and is reported as a finding, not hidden.

**Test set (400 held-out rows, scored via the live endpoint), independent of the metric
SageMaker reported during training:**

| Metric | Value |
|---|---|
| AUROC | 0.7486 |
| AUPRC | 0.5527 (prevalence baseline 0.2975) |

**Calibration** (mean predicted vs actual event rate by decile): the model is
under-confident in the bottom half (e.g. decile 4: predicted 0.034 vs actual 0.250) and
somewhat over-confident at the very top (decile 10: predicted 0.879 vs actual 0.750).
Ranking is monotonic and reasonable; absolute probability calibration is not production
grade. Full table: [`evidence/calibration-deciles.csv`](evidence/calibration-deciles.csv).

**Subgroup fairness check (AR-01 / RMF-MS-2), the reason the disadvantage variable was
held out in the first place:**

| Group | n | Mean predicted risk | Actual event rate | Gap | AUC | Recall @ 0.30 |
|---|---|---|---|---|---|---|
| Not disadvantaged | 289 | 0.2329 | 0.2491 | 1.6 pts | 0.7335 | 0.514 |
| Disadvantaged | 111 | 0.2754 | 0.4234 | **14.8 pts** | 0.7753 | 0.511 |

**Finding:** ranking (AUC) and recall at a fixed threshold are balanced across the two
groups, so a simple "is this group flagged as often" check would look fine. But the model
**systematically understates absolute risk for the disadvantaged group by roughly nine times
the gap seen elsewhere**, because it cannot see the factor driving their extra true risk.
Since the risk **tier** (low/moderate/high) is set from the absolute score, this group is
measurably more likely to land in a lower tier than their true risk warrants, and therefore
to receive less proactive outreach. This is a concrete, measured instance of risk **AR-01**,
not a hypothetical one, and it is exactly the failure mode that excluding the sensitive
attribute as a model input does **not**, by itself, prevent. Feeds directly into treatment
gap **G-02**.

## Cost

- Training: `ml.m5.large` for 124 seconds, a fraction of a cent.
- Hosting: `ml.t2.medium` real-time endpoint, up for approximately 12 minutes end to end
  (create, invoke, delete), a few cents.
- S3, KMS, data transfer: negligible at this data volume.
- **Total spend: well under $1** against the ~$100 AWS credit balance. No separate budget
  alert was created for this short-lived build; the existing `meridian-lab-monthly` alert
  from the Cloud Risk Assessment lab covers the same account.

## Teardown confirmation

Completed 2026-09-25. In order: endpoint deleted and confirmed gone (`describe-endpoint`
returned "Could not find endpoint"), endpoint config deleted, model deleted, S3 bucket
emptied and deleted, KMS key `febe85f3-860f-453c-bd0f-1cc21e600c06` scheduled for deletion
(7-day minimum window, deletes 2026-10-02), execution role `meridian-ai-sagemaker-role`
detached, its inline policy removed, and the role deleted, confirmed via a final
`get-role` call returning `NoSuchEntity`. Nothing from this build remains running or
billing.
