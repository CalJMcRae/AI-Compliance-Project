# Minimal AWS Model: Build Notes and Evidence

The smallest real footprint that produces genuine technical-control evidence. Build it,
capture the evidence, tear it down.

## Plan

- [x] Synthetic training dataset (reuse the Cloud Risk Assessment lab data pattern)
- [x] S3 bucket `meridian-ai-data-<suffix>`, KMS encrypted, Block Public Access on
- [x] Execution role `meridian-ai-sagemaker-role` created
- [ ] SageMaker built-in XGBoost training job (small instance, spot if available) — **blocked, see below**
- [ ] Deploy to Serverless Inference endpoint (scales to zero)
- [ ] Scoped execution role: read the one bucket, write scores, nothing else
- [ ] Enable model data capture
- [ ] Confirm CloudTrail is recording (shared with the Cloud Risk Assessment account)
- [ ] Run a few hundred inferences from a short script; save sample input and output
- [ ] Capture evidence (screenshots plus CLI output) into `../docs/screenshots/`
- [ ] Record metrics: AUROC, AUPRC, calibration, subgroup performance
- [ ] Tear down: delete endpoint, endpoint config, model; empty and delete the bucket

## Build log

- **Dataset:** synthetic, 2,400 rows (1,600 train / 400 validation / 400 test), 12 features
  (utilisation, lab-value trends, condition flags, age band, sex), no demographic proxies
  per system description decision B. One additional variable, a socioeconomic/access
  disadvantage flag, was generated to genuinely elevate a subgroup's true risk (42.3% true
  event rate vs 24.9% for everyone else in the test set) but deliberately **excluded** as a
  model input, so it can be used later to run a real subgroup fairness check (AR-01,
  RMF-MS-2): does the model under-predict for this subgroup precisely because it cannot see
  what is driving their extra risk.
- **S3 bucket:** `meridian-ai-data-328064416121`, KMS-encrypted (`alias/meridian-ai`, key
  rotation enabled), Block Public Access on all four settings, tagged `project=meridian-ai`.
- **Execution role:** `meridian-ai-sagemaker-role`, trust limited to `sagemaker.amazonaws.com`.
  Attached `AmazonSageMakerFullAccess` (a documented scoping trade-off, see below) plus an
  inline policy scoping S3 access to only the `meridian-ai-data-*` bucket.
- **Scoping trade-off recorded:** the SageMaker execution role uses the AWS-managed
  `AmazonSageMakerFullAccess` policy rather than a hand-scoped one. Hand-scoping SageMaker's
  own training/hosting permissions reliably is a real time sink, and this is a role only the
  SageMaker service can assume, for a build that is torn down after evidence capture. The
  compensating control is the inline S3 policy, which does scope data access to the single
  bucket regardless of what the managed policy would otherwise allow.
- **KMS key policy note:** IAM roles must exist before they can be added as principals in a
  KMS key policy, so the key was policy-updated in two passes (root and `Callum-v2` first,
  the execution role added once it existed).

### Blocked: SageMaker training instance quota

`CreateTrainingJob` fails account-wide with `ResourceLimitExceeded` on every instance type
tried (`ml.m5.large`, `ml.m4.xlarge`, `ml.m5.xlarge`, `ml.c5.xlarge`), each reporting a
service quota of **0 instances** for training job usage. This is an AWS default anti-abuse
gate on lightly-used accounts, not a permissions or configuration error, `Callum-v2`'s
policy is otherwise sufficient (confirmed: the API call reaches quota evaluation, it does
not fail on authorization). A Service Quotas increase request has been submitted for
`ml.m5.large for training job usage` (quantity 1). Resuming once approved.

## Evidence captured

`TODO`, pending the quota approval and training run.

## Cost

`TODO` record actual spend against the $100 credit balance. Budget alert: `TODO`.

## Teardown confirmation

`TODO` date, plus a describe call showing the endpoint no longer exists.
