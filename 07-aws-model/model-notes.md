# Minimal AWS Model: Build Notes and Evidence

The smallest real footprint that produces genuine technical-control evidence. Build it,
capture the evidence, tear it down.

## Plan

- [ ] Synthetic training dataset (reuse the Cloud Risk Assessment lab data pattern)
- [ ] S3 bucket `meridian-ai-data-<suffix>`, KMS encrypted, Block Public Access on
- [ ] SageMaker built-in XGBoost training job (small instance, spot if available)
- [ ] Deploy to Serverless Inference endpoint (scales to zero)
- [ ] Scoped execution role: read the one bucket, write scores, nothing else
- [ ] Enable model data capture
- [ ] Confirm CloudTrail is recording (shared with the Cloud Risk Assessment account)
- [ ] Run a few hundred inferences from a short script; save sample input and output
- [ ] Capture evidence (screenshots plus CLI output) into `../docs/screenshots/`
- [ ] Record metrics: AUROC, AUPRC, calibration, subgroup performance
- [ ] Tear down: delete endpoint, endpoint config, model; empty and delete the bucket

## Evidence captured

`TODO`

## Cost

`TODO` record actual spend against the $100 credit balance. Budget alert: `TODO`.

## Teardown confirmation

`TODO` date, plus a describe call showing the endpoint no longer exists.
