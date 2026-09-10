# EU AI Act Risk Classification: Meridian Rising Risk Score

The classification decision, the reasoning, and the points where the regulation is genuinely
unclear. Written to be challenged.

**Regulation:** EU AI Act (Regulation (EU) 2024/1689), as amended by the Digital AI Omnibus
(in force 27 July 2026).
**Assessment position:** the Meridian Rising Risk Score (MRRS) is treated as **high-risk**.
The reasoning, and the weaker contrary reading, are both set out below.

This analysis rests on the following facts from
[`../01-ai-system-definition/system-description.md`](../01-ai-system-definition/system-description.md).
If any of them change, the classification must be revisited:

- MRRS predicts near-term (90-day) health risk for named patients from their clinical data.
- Its output ranks patients for proactive clinician outreach and care-management enrolment.
- A clinician makes every care decision; MRRS contacts no one and changes no record.
- It is not emergency triage. It runs as a nightly batch over an outpatient panel.
- It is operated by private partner clinics, not a public authority.

---

## 1. Is it an "AI system" (Art. 3(1))?

Yes. It is machine-based, it infers from inputs (clinical features) how to generate an output
(a risk score and tier), and that output influences a real environment (which patients a
clinician contacts first). No further argument needed.

## 2. Is it a prohibited practice (Art. 5)?

No. Walking the Art. 5 list:

- **Social scoring (5(1)(c))** is the closest. It is not met: MRRS is a bounded,
  purpose-specific clinical-adjacent tool, not a general-purpose score of a person's
  trustworthiness or behaviour used to their detriment in unrelated contexts.
- Not subliminal or manipulative technique; not exploitation of vulnerability; not
  untargeted facial scraping; not emotion recognition in work or education; not biometric
  categorisation by sensitive attributes; not real-time remote biometric identification.

**Conclusion:** not prohibited.

## 3. Is it high-risk?

There are two independent routes into high-risk. Only one needs to succeed.

### Route A: safety component of, or itself, a regulated product (Art. 6(1), Annex I)

This route asks whether MRRS is a **medical device** under the Medical Device Regulation
(Regulation (EU) 2017/745, MDR) that also requires third-party conformity assessment.

- **Argument that it is a device.** The MDR covers software intended for a medical purpose
  such as prediction or prognosis of disease. MRRS processes individual patient data to
  produce patient-specific information (a probability of a near-term care event) used for a
  clinical purpose (deciding who needs proactive care). MDCG 2019-11-style qualification
  guidance tends to pull software of this kind into "medical device", and most
  clinical-decision-support software lands at Class IIa or above, which requires notified-body
  involvement.
- **Argument that it is not.** MRRS does not diagnose, does not recommend a treatment, and
  does not act. It orders an administrative work queue. The clinical decision is entirely
  downstream and human. A provider could argue the "medical purpose" threshold is not met.

**This is genuinely contested and needs a formal MDR qualification opinion from regulatory
counsel.** It is flagged as an open question in section 6. The high-risk conclusion below
does not depend on resolving it, because Route B is sufficient on a purposive reading.

### Route B: Annex III use case (Art. 6(2))

The candidate is **Annex III point 5**, "access to and enjoyment of essential private
services and essential public services and benefits."

**On the black-letter text, the fit is imperfect.** The enumerated sub-points are:

| Sub-point | Covers | Fit with MRRS |
|---|---|---|
| 5(a) | Public authorities evaluating eligibility for, or granting / reducing / revoking, essential public assistance benefits and services including healthcare | No. MRRS is run by private clinics and is not an eligibility gate. |
| 5(c) | Risk assessment and pricing for life and health insurance | No. |
| 5(d) | Emergency healthcare patient triage; dispatch prioritisation of emergency first response | No. MRRS is non-urgent, prospective, panel-level. |

A strict-textualist provider could therefore argue MRRS is **not** listed in Annex III as it
currently stands, and so is not high-risk under Route B.

**On a purposive reading, and consistent with the regulator's stated direction, it is in
scope.** The chapeau of point 5 is about *access to and enjoyment of* essential services. A
tool that ranks which patients receive proactive care management materially affects their
access to and enjoyment of a healthcare service. The European Commission's draft guidelines
on high-risk classification (Art. 6(5); consultation closed 23 July 2026, not yet adopted)
expressly treat patient-prioritisation tools as high-risk, citing an emergency-department
prioritisation system as high-risk "even without performing clinical assessment in the formal
sense." MRRS prioritises patients on a longer horizon but does the same kind of thing.

**The profiling point is decisive once inside Annex III.** Article 6(3) lets an Annex III
system escape high-risk if it performs only a narrow procedural or preparatory task and does
not materially influence decisions. That derogation is **switched off entirely where the
system profiles natural persons** (Art. 4(4) GDPR: automated processing to analyse or predict
aspects concerning a person's health). MRRS does exactly that. So if MRRS is an Annex III
system, it is high-risk regardless of the "a human decides" design.

### Classification decision

**MRRS is treated as high-risk.** The reasons, in order of weight:

1. On a purposive reading of Annex III point 5, aligned with the Commission's draft
   classification guidance, a patient-prioritisation tool affecting access to essential
   healthcare is in scope; and once in Annex III, profiling removes any Art. 6(3)
   derogation.
2. The MDR route (Route A) is arguable and, if it applies, independently mandates high-risk
   status.
3. Even where the black-letter classification is contestable today, MRRS is a
   risk-prediction tool sold into healthcare. Meridian's customers will run vendor due
   diligence and expect AI Act readiness. Building the high-risk control set now is far
   cheaper than retrofitting it, and the regulatory direction of travel is one-way.

### The contrary reading, stated honestly

A well-advised provider could argue that, as Annex III point 5 is currently drafted, a
non-emergency outpatient risk-stratification tool for care management is not enumerated, and
that MRRS is therefore not high-risk, carrying only limited-risk transparency duties. This
reading is legally available today. It is fragile: it depends on the guidelines not being
finalised as drafted, on the MDR question resolving in the provider's favour, and on a
narrow reading of "access to essential services." **This assessment does not rely on it.**

## 4. Limited-risk transparency (Art. 50) and adjacent duties

- **Art. 50** transparency obligations are narrow (AI interaction disclosure, synthetic
  content marking, emotion recognition, deepfakes). A back-office batch score that patients
  never interact with likely does not trigger Art. 50 directly.
- **GDPR Art. 22** (solely automated decisions with legal or similarly significant effect):
  not engaged, because a clinician makes the care decision. If the clinical workflow ever
  lets the score auto-enrol or auto-exclude a patient, Art. 22 must be re-checked.
- **GDPR Arts. 13 to 14** transparency about the processing, and the question of whether
  patients should be told an AI system informs their care prioritisation, remain live
  regardless of the AI Act classification. Tracked as risk **AR-07** and control **AIA-05**.

## 5. What follows from a high-risk classification

The provider (Meridian) obligations that the control catalogue in
[`../03-control-catalogue/`](../03-control-catalogue/) is built around:

| Article | Obligation | Catalogue ID |
|---|---|---|
| Art. 9 | Risk management system across the lifecycle | AIA-01 |
| Art. 10 | Data and data governance, including bias examination | AIA-02 |
| Art. 11 + Annex IV | Technical documentation | AIA-03 |
| Art. 12 | Automatic logging over the system lifetime | AIA-04 |
| Art. 13 | Transparency and instructions for deployers | AIA-05 |
| Art. 14 | Human oversight by design | AIA-06 |
| Art. 15 | Accuracy, robustness, cybersecurity | AIA-07 |
| Art. 17 | Quality management system | AIA-08 |
| Art. 43 | Conformity assessment before placing on the market | (add to catalogue) |
| Art. 72 | Post-market monitoring | AIA-09 |

## 6. Where the regulation is unclear (open questions)

1. **MDR qualification.** Is MRRS a medical device? Needs a formal regulatory opinion. Drives
   Route A and the conformity-assessment path.
2. **Final Art. 6(5) guidelines.** The Commission's high-risk classification guidelines were
   in consultation to 23 July 2026 and are not yet adopted. The classification here assumes
   they land broadly as drafted.
3. **Meaning of "access to and enjoyment of essential services"** in Annex III point 5 for a
   private-sector prospective tool. No settled interpretation yet.
4. **Member-state overlay.** A specific deployment may engage national health-AI or
   medical-software rules not assessed here.

## 7. Timeline note

High-risk obligations for Annex III systems now apply from **2 December 2027** (deferred from
2 August 2026 by the Digital AI Omnibus, in force 27 July 2026). Product-embedded (Annex I)
systems: 2 August 2028. This is a deferral, not a cancellation, and the date has already
moved once. The recommendation is to build the control set ahead of the deadline rather than
wait.
