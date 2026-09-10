# Meridian Rising Risk Score: Data Flow

```mermaid
flowchart LR
    subgraph Clinic["Partner clinic"]
        EHR[(EHR)]
        DEID[De-identify + extract]
        DASH["Clinician dashboard<br/>ranked outreach list"]
        CLIN(["Clinician<br/>decides whether to act"])
    end

    subgraph AWS["AWS - project=meridian-ai"]
        S3[("S3 extract bucket<br/>KMS encrypted")]
        JOB["Nightly scoring job"]
        EP["SageMaker Serverless<br/>Inference endpoint<br/>XGBoost"]
        STORE[("Scores datastore")]
        CT["CloudTrail<br/>invoke + data access logs"]
    end

    EHR --> DEID --> S3 --> JOB --> EP --> JOB --> STORE --> DASH --> CLIN
    CLIN -.outreach decision.-> EHR
    JOB -. logs .-> CT
    EP -. logs .-> CT
```

## Trust boundaries

| Boundary | What crosses it | Control |
|---|---|---|
| Clinic EHR to AWS | De-identified feature extract only | De-identification at source; TLS in transit; KMS at rest |
| Scoring job to endpoint | Feature vectors | Scoped IAM role; VPC / private networking `TODO` |
| Scores back to dashboard | Score, tier, top features | `TODO` |

## Affected-person touchpoint

Patients never interact with the system. The only effect on a patient is a change in the
*likelihood and priority* of proactive outreach, mediated entirely by a clinician's
decision.
