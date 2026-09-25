# Understanding the AI Compliance Assessment

A section-by-section walkthrough of the project, written so you can explain any part of it
out loud: what it is, why it was built that way, and what it found. Companion to the
[decision log](../../GRC%20AWS%20Project/docs/decision-log.md) style document for the AWS
lab project.

---

## The one-sentence version

Assessed a simulated clinical AI system against the EU AI Act and NIST AI RMF, classified it
as high-risk through a genuinely contested regulatory question, built the whole assessment
inside a real GRC platform (Eramba), then built and evaluated a real machine-learning model
on AWS to test the assessment against actual evidence, which confirmed a critical fairness
finding with measured numbers and surfaced a new one a paper-only review would have missed.

## The scenario

Meridian Health Analytics (the same fictional company as the AWS lab project, for narrative
continuity) builds the **Meridian Rising Risk Score (MRRS)**: a model that scores each
patient in a partner clinic's panel for near-term health risk, so clinicians know who to
proactively reach out to. A clinician decides whether to act; the model only recommends. It
is trained on de-identified clinical data and intended to run on AWS.

Why this scenario: it is genuinely ambiguous under the EU AI Act (not obviously high-risk,
not obviously exempt), it has a real fairness angle worth testing (patient prioritisation
naturally raises equity questions), and it connects back to the AWS lab project rather than
starting a disconnected narrative.

---

## Section 1 — System definition (`01-ai-system-definition/`)

**What it is.** The technical documentation of MRRS: what it does, who uses it, what data
goes in, what comes out, and a set of governance decisions that had to be made before
anything else could proceed.

**The decisions worth being able to explain:**

- **Patients are told an AI system is used**, through the partner clinic's privacy notice,
  even though the EU AI Act's own transparency article (Art. 50) probably would not have
  required it for a back-office tool like this. Chosen anyway because GDPR's own
  transparency articles (13-14) point the same way, and it costs little to be upfront.
- **No demographic proxies (race, language, deprivation) as model inputs.** Age and sex are
  kept as legitimate clinical risk factors; the rest were excluded on the theory that
  training on them risks baking in existing care-access disparities for little predictive
  gain. This decision is the reason the fairness finding later in the project could even be
  measured, a variable representing exactly that kind of disadvantage was generated but
  deliberately kept out of the model, specifically so a subgroup check could be run against
  ground truth the model never saw.
- **A production gate**: a defined go/no-go review with named conditions, not just "we'll
  know it when we see it."

**How to talk about it:** this section exists because the EU AI Act's Annex IV technical
documentation requirement and the NIST AI RMF's Map function both want the same thing
answered first, "what exactly is this system, for whom, and what happens if it is wrong",
before you can meaningfully assess anything else.

---

## Section 2 — EU AI Act classification (`02-eu-ai-act-classification/`)

**What it is.** The single most important judgement call in the project: is MRRS high-risk
under the EU AI Act, and why.

**The honest answer: it is not a clean fit on the literal text.** Annex III point 5 (the
category that would cover "access to essential services") lists things like emergency
triage and public-benefit eligibility decisions. MRRS is neither, it is a private company's
non-emergency, prospective outreach-prioritisation tool. A strict reading could argue it out
of scope entirely.

**Why it is still treated as high-risk anyway, two independent reasons:**

1. **Purposive reading.** The European Commission's own draft guidance treats
   patient-prioritisation tools as high-risk even when they are not formal clinical
   assessments, because they affect access to care. And once a system is inside Annex III at
   all, a legal carve-out (Article 6(3)) that would otherwise let it escape high-risk status
   is switched off entirely if the system profiles people, which MRRS does (it processes
   personal data to predict something about a person's health). That single fact makes the
   "narrow procedural task" escape route unavailable regardless of the human-in-the-loop
   design.
2. **The medical device question.** Software that predicts a health outcome to drive a
   clinical action might independently qualify as a medical device under EU law (the MDR),
   which would also make it high-risk. This is a genuinely open legal question, referred to
   counsel rather than resolved here.

**Why commit to high-risk when the answer is genuinely unclear:** because the cost of being
wrong runs one direction. Building to the higher standard and being told later it was not
strictly required costs some extra work. Not building to it and being told it was required
costs a redo under regulatory pressure, with customers who will ask about AI Act readiness
regardless of how a court might eventually rule on Annex III's exact wording.

**One more fact worth knowing:** the deadline for these obligations was pushed from August
2026 to December 2027 by an EU simplification package partway through this project, deferred,
not cancelled. That is itself part of the answer to "why not just wait and see", the deadline
has already moved once.

---

## Section 3 — Control catalogue (`03-control-catalogue/`)

**What it is.** 18 specific obligations the classification in Section 2 creates, turned into
a checklist with an owner and a status each: 12 from the EU AI Act itself (risk management,
data governance, technical documentation, logging, transparency, human oversight, accuracy,
quality management, post-market monitoring, conformity assessment, registration, deployer
support), 5 from the NIST AI RMF (which is voluntary but maps well onto the same obligations
and adds a bias-measurement discipline the AI Act states but does not spell out in detail),
and 1 placeholder for an ISO/IEC 42001 management-system check, deliberately run last.

**How to talk about it:** this is the bridge between "the law says X" and "here is the actual
work list." Two frameworks were combined rather than picked one, because the EU AI Act tells
you *what* is required and NIST AI RMF is better at telling you *how* to actually run the
underlying risk-management discipline (their MEASURE function in particular).

---

## Section 4 — AI risk register (`04-ai-risk-register/`)

**What it is.** Ten specific things that could go wrong, each scored for likelihood and
impact (1 to 5 each), the same scoring convention used across the whole GRC portfolio, so
results are comparable project to project.

**The two you should know cold:**

- **AR-01, Critical.** The model could systematically under-score an already under-served
  group of patients, meaning they get less proactive outreach, compounding an existing
  disparity. This was the headline hypothesis before any code was written.
- **AR-03, Critical.** Clinicians could start trusting the ranked list so much that patients
  it does not flag stop getting a second look, "automation bias."

**AR-10 is the interesting one, because it was not on the original list.** It was added
after the AWS build (Section 7) revealed the model overfits and is poorly calibrated,
something no amount of design review could have found without actually training a model.
That is the whole argument for doing Section 7 at all.

**How to talk about it:** every risk here has a plausible mitigation path already scoped in
the treatment plan (Section 6), and none are scored down to "Low" as a target even after
mitigation, a fairness or oversight risk in a clinical system is treated as something you
manage continuously, not something you fix once and close.

---

## Section 5 — Assessment and gaps (`05-assessment-and-gaps/`)

**What it is.** A snapshot of where all 18 controls actually stand, mirrored from the live
Eramba instance: zero fully compliant (expected, the system is pre-production), seven
partially implemented, eleven not started.

**The detail worth remembering:** four of those seven "partial" controls only earned that
status because the AWS build actually ran the evaluation they call for and found real
problems, they are "partial" in the sense of "evaluated, not yet fixed," not "in progress."
That distinction matters if someone asks whether "partial" means good news or bad news, here
it is mostly the latter, honestly reported.

---

## Section 6 — Treatment plan (`06-treatment-plan/`)

**What it is.** The 18 open control gaps, consolidated into 11 concrete pieces of work
(several controls close under one fix, the same "don't list 18 disconnected line items"
instinct used in the AWS project's remediation plan), phased into 0-30, 30-90 and 90+ day
windows by urgency and dependency.

**Worth knowing:** the mitigation for the confirmed fairness finding (gap G-02) had its
priority window pulled forward from 30-90 days to 0-30 days once the AWS build turned it from
a hypothesis into a measured fact. That is a live example of a risk-based prioritisation
actually reacting to new evidence, not just a document written once and left alone.

---

## Section 7 — The AWS model build (`07-aws-model/`)

**What it is, and why it matters most.** A real, working, minimal version of MRRS: a
synthetic dataset, a real SageMaker training job, a real deployed endpoint, real predictions
scored against real held-out data, then everything torn down the same day. This is the part
of the project that turns "we assessed the design" into "we tested it."

**The one number to remember:** for a deliberately held-out variable representing patient
disadvantage, never shown to the model, the model's predicted risk undershot the group's true
event rate by about **15 percentage points**, versus roughly **1.5 points** for everyone
else, even though the model's ranking ability (AUC) and its recall at a fixed threshold were
about equal across both groups. In plain terms: a simple check ("does the model flag this
group often enough") would have looked fine. Looking at the actual risk *level* the model
assigns them tells a different story, and since the system's low/moderate/high tier is set
from that level, the disadvantaged group is more likely to land in a lower tier than their
true risk justifies.

**The second finding**, model overfitting (near-perfect accuracy on the training data,
meaningfully worse on data it had not seen), was not something anyone was looking for, it
came out of just reading the training job's own metrics honestly instead of only checking
the number that looked good.

**What actually went wrong along the way, useful colour for an interview:**

- AWS defaults new SageMaker training capacity to **zero instances** on lightly-used
  accounts. Had to request a quota increase mid-build (approved the same day).
- A custom encryption key needs one extra, easy-to-miss permission
  (`kms:CreateGrant`) beyond the obvious encrypt/decrypt ones before SageMaker can actually
  use it.
- The specific container image path for the algorithm had changed since the version widely
  documented online, the older path only serves a legacy version.
- The local command-line tooling was old enough to predate a newer AWS deployment option, so
  a small always-on endpoint was used instead of a scale-to-zero one, torn down immediately
  after, and documented as a tooling limitation rather than pretended away.

None of these were permission mistakes or bad design, they were exactly the kind of small,
specific frictions that show up when you actually build something instead of only describing
it, and every one is logged with the fix, not just the outcome.

---

## Why Eramba (the GRC platform)

Everything above could have lived in spreadsheets and documents alone. It was also built
inside **Eramba**, a real open-source GRC platform, self-hosted for free, because a hiring
manager reading "GRC tools: Eramba" gets something a document cannot show: that the risks,
controls, and compliance status are linked to each other and queryable, not just described
in prose next to each other. The controls, risk register, compliance mapping and gap analysis
all live there, cross-referenced, exactly like they would in a real organisation's GRC
tooling, just at a smaller scale.

---

## How conclusions were reached, the through-line

The same discipline ran through every section:

- **State the assumption, make the trade-off explicit, acknowledge the residual risk**, the
  same standard used across the whole portfolio.
- **Prefer measured evidence over design intent wherever it was possible to get it**, which
  is the entire reason Section 7 exists instead of stopping at Section 6.
- **Surface the contested judgement calls rather than hide them.** The classification
  section states the weaker reading it does not rely on. The risk register notes which
  scores are close calls. Nothing here pretends to more certainty than it has.
- **Revise when new evidence arrives.** AR-01 kept its score but was reframed from
  hypothesis to measured fact. AR-10 was added because it was found, not because it was
  planned for.

## Limitations, said plainly

- All data, including the disadvantage variable central to the fairness finding, is
  synthetic. The *pattern* it revealed is a real and general failure mode; the specific
  numbers are not evidence about any real population.
- The AWS build was a minimal proof of concept, one training run, no hyperparameter search,
  no adversarial testing yet. Real, but not production-grade.
- Single assessor throughout. A real engagement would have a second reviewer, especially on
  the medical-device question.
- The regulation itself is still moving (draft guidance, a deadline that already shifted
  once), so the classification is the best current reading, not a settled one.

---

## If you are asked about this project, the likely questions

**"Why treat it as high-risk when the text does not clearly say so?"**
Because the purposive reading and the regulator's own direction point that way, because
profiling removes the escape route if it is in scope at all, and because the cost of being
wrong is asymmetric, over-building costs effort, under-building costs a redo under pressure.

**"What was the actual finding, in one sentence?"**
The model's ranking and flagging behaviour looked fair across groups, but its absolute risk
scores understated true risk almost ten times more for a disadvantaged subgroup than for
everyone else, which matters because that absolute score sets the care-priority tier.

**"Why build a real model instead of just documenting the design?"**
Because the overfitting finding and the precise size of the fairness gap were both things a
paper review could not have found, they only existed once a real model was trained and
tested against real held-out data.

**"What would you do differently?"**
Use a time-based train/test split instead of a random one (closer to how the system will
actually be used, and would test for drift), and get a second reviewer on the medical-device
question before treating the classification as final.
