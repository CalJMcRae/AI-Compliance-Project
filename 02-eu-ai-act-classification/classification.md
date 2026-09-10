# EU AI Act Risk Classification: Meridian Rising Risk Score

The classification decision, the reasoning, and the points where the regulation is genuinely
unclear. This is the intellectual core of the project; it is written to be challenged.

---

## 1. Is it an "AI system" under Art. 3(1)?

`TODO` short answer yes, with the one-line reason (machine-based, infers from input to
produce a prediction that influences an environment).

## 2. Prohibited practice (Art. 5)?

`TODO` walk the Art. 5 list. Expected: no. Note the closest call (social scoring style
concerns) and why MRRS is not that.

## 3. High-risk (Art. 6 + Annex III)?

Two routes to high-risk. Both need to be worked through:

### Route A: safety component of a regulated product (Art. 6(1), Annex I)

`TODO` Is MRRS a medical device or an accessory under the Medical Device Regulation
(EU 2017/745)? Software that provides information used for decisions with a medical purpose
can be a device. Argument for: it informs clinical prioritisation. Argument against: it does
not diagnose, and prioritising outreach may fall short of a "medical purpose" in the MDR
sense. **This is the central ambiguity.** Record both readings and which one this assessment
proceeds on, and why.

### Route B: Annex III use case

`TODO` walk the Annex III list. Candidates:
- 5(a) evaluation of eligibility for public assistance benefits and services: probably not,
  MRRS is not an eligibility gate.
- 5(b) / essential private services and healthcare access: closer. Does prioritising
  outreach "materially affect access to essential healthcare services"? Argue it.

## 4. Limited-risk transparency (Art. 50)?

`TODO` If not high-risk, do any transparency duties still apply (informing people they are
subject to an AI system)? Consider the affected-person notice question from the system
description.

## 5. Classification decision

`TODO` state the outcome (high-risk / limited-risk / minimal), the route that drives it, and
the confidence level. If high-risk, the obligation mapping in
[`../03-control-catalogue/`](../03-control-catalogue/) follows from here.

## 6. Where the regulation is unclear

`TODO` list the open questions and what guidance would resolve them (Commission guidance on
Annex III(5), the MDR / AI Act interface, the meaning of "materially affect access").
