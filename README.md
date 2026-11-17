# ClaimSense AI

**A claims adjudication co-pilot for a motor insurer, built as a five-week forward-deployed engineering (FDE) engagement.**

Author: **Anju Mercian**
Status: **Milestone 1 of 5 complete (Discovery and scoping)**

![Milestone](https://img.shields.io/badge/milestone-1%20of%205-blue) ![Data](https://img.shields.io/badge/data-synthetic%20only-green) ![Python](https://img.shields.io/badge/python-3.12%2B-informational)

---

## The problem

Adjusters at the client, a motor insurer, handle every claim by hand across three systems: **claims management**, **policy administration** and a **document store**. One adjuster gathers the file, checks coverage, assesses fraud, total-loss and injury exposure, and writes the first decision. This first pass takes **hours** per claim.

Three things make it worse:

- **Integration friction.** The evidence for one claim is spread across three systems with no single view, so most of the first pass goes on gathering rather than judging.
- **Unrecorded reasoning.** Decisions vary between adjusters, and nobody writes down why, so outcomes are inconsistent and hard to defend.
- **Threshold-driven fraud referral.** Claims go to the Special Investigations Unit (SIU) when a fraud score crosses a threshold, not because someone read the file and found a problem.

Adding more rules to the existing rule table won't fix this. Coverage depends on the **wording of a policy clause**, and fraud depends on whether **the story stays consistent across documents**. Both of those are reading tasks.

## What ClaimSense does

Given a claim, ClaimSense returns:

1. a **first decision** (one of `APPROVE`, `DENY`, `REFER_SIU`, `ESCALATE`),
2. the **reasoning** behind it, and
3. the **evidence** the reasoning rests on, with citations to the policy clauses it actually retrieved.

If the user isn't entitled to the claim, ClaimSense **refuses the request, returns no claim data, and records the refusal in the audit trail**. It is a co-pilot: **a licensed adjuster confirms every outcome**, and ClaimSense never finalises a decision on its own.

### Fixed constraints

| # | Constraint | What it means in practice |
|---|---|---|
| 1 | **Deterministic operations are code** | Identity, access control, policy dates and money calculations are plain Python. A model never handles them. |
| 2 | **Judgment is delegated to a model, and cited** | Clause interpretation, fraud reading and the draft decision come from the model, and every judgment states its evidence. |
| 3 | **Synthetic data only** | No production or personally identifiable data at any stage. |

```
 Claim, documents,         ┌──────────────────────────┐
 policy, history  ───────► │  Deterministic code      │  identity · access control
                           │                          │  policy dates · money
                           ├──────────────────────────┤
                           │  Model judgment (cited)  │  read the file · coverage
                           │                          │  risk · draft decision
                           └────────────┬─────────────┘
                                        ▼
                     First decision + reasoning + citations
                                        ▼
                     Licensed adjuster confirms every outcome
```

### Technology standard

| Capability | Standard |
|---|---|
| Typed, verifiable model output | OpenAI Agents SDK |
| Retrieval with source attribution | Searchable policy-clause corpus |
| Controlled access to client systems | Self-built data-access service with authentication |
| Portable execution | Docker containers |
| Change control | CI with an evaluation gate |
| Hosting, observability, recovery | AWS |

Runtime is Python 3.12+. The system will also need Node 20+, Docker, an AWS account and an OpenAI API key.

---

## Engagement roadmap

| Week | Milestone | Status |
|---|---|---|
| 1 | **Discovery and scoping** | ✅ **Complete** |
| 2 | Statement of work, pricing and architecture | ⏳ Not started |
| 3 | Production agents | ⏳ Not started |
| 4 | Integration and access control | ⏳ Not started |
| 5 | Evaluation, observability, reliability and handover | ⏳ Not started |

---

## Milestone 1: Discovery and scoping ✅

**Goal:** find out what the client actually needs, and agree what success will be measured against, before writing any agent code (see `ADR-001`).

### Steps taken

#### Step 1: Clarify
I drew up a set of discovery-workshop questions to get from the client's symptom ("first pass takes hours") to what is actually causing it. The questions start with *"Show me how you do a first pass today"* and cover:
- where the time goes,
- how many systems an adjuster uses per claim,
- how ambiguous policy wording gets resolved,
- what triggers an SIU referral,
- how disagreements between adjusters get settled,
- current baselines for handling time and referral rate.

#### Step 2: Decompose
I broke the adjuster's first pass into five stages and labelled each as either deterministic code or model judgment:

| Stage | What happens | Nature |
|---|---|---|
| Gather the file | Pull claim, policy and documents from 3 systems | Deterministic (integration) |
| Check coverage | Policy in-force dates + clause wording | Dates = code, wording = model |
| Assess fraud | Narrative consistency across documents | Model judgment (cited) |
| Total loss / injury | Estimate vs value, injury indicators | Money = code, indicators = model |
| Write decision | Draft first decision + reasoning | Model draft, adjuster confirms |

#### Step 3: Scope
**Working hypothesis:** the bottleneck that limits adjuster throughput is *gathering the file*: evidence scattered across three systems. Unrecorded reasoning makes it worse by causing inconsistent decisions. The time split behind this hypothesis is a placeholder, to be replaced with a time study during discovery.

#### Step 4: Write the engagement charter
A one-to-two-page charter covering the problem statement, the three client systems and how ClaimSense will connect to each, the fixed constraints, what's in and out of scope, and how success is defined.

| System | Integration point | Access |
|---|---|---|
| Claims management | REST read of claim by ID. A repeat-safe (idempotent) write-back of the draft decision. | Read + one guarded write |
| Policy administration | REST read of policy. Clause wording exported into the search corpus. | Read only |
| Document store | List and fetch documents by claim ID, with text extracted | Read only |

I also built an **integration inventory** that rates each system on two kinds of difficulty: **technical** and **political**. Access to data is negotiated with people, not just requested through an API.

#### Step 5: Map stakeholders and RACI
I mapped everyone who can approve, review, use or block the co-pilot, using RACI: who is **R**esponsible, **A**ccountable, **C**onsulted and **I**nformed. Exactly one person is Accountable. The four roles the brief requires are all covered:

| Brief role | Stakeholder |
|---|---|
| Approver | Head of Claims (Accountable) |
| Reviewer of co-pilot output | Claims QA Lead / Senior Adjuster |
| Daily user | Licensed motor adjusters |
| Can block rollout | CISO / InfoSec, Compliance, and the adjusters themselves |

The adjusters' veto is a **quiet** one: they can sink the rollout without any formal say, simply by working around a tool they don't trust.

#### Step 6: Set quantified success criteria
Each criterion is written as `[metric] [threshold] [population]`, specific enough to settle any later dispute about whether the engagement worked. A built-in scorer checks that every criterion is measurable.

| Dimension | Proposed criterion |
|---|---|
| Decision accuracy | ≥ 85% match with the gold label on the held-out eval suite, and ≥ 75% on each outcome class |
| Pipeline latency | p95 ≤ 120 s and p50 ≤ 60 s per claim, on the deployed stack |
| Adjuster confirmation rate | ≥ 80% of first decisions confirmed unchanged on pilot claims |
| Fraud routing accuracy | `REFER_SIU` recall ≥ 90%, with false referrals ≤ 10% of non-fraud cases |

**Guardrails (zero tolerance):**
- 0 claim data returned to unentitled users
- 0 decisions finalised without an adjuster
- 0 citations to clauses that were never retrieved

> Thresholds are *proposed* and need sign-off from the Head of Claims before Milestone 2.

#### Step 7: Record architectural decisions in `DECISIONS.md`
I opened a decision log in a lightweight architecture-decision-record format, with seven dated decisions. Each one records the **decision, why it was made, the alternatives rejected, the owner, and what should trigger a revisit**.

| ID | Decision |
|---|---|
| ADR-001 | Week 1 is discovery only; no agent code until success criteria are agreed |
| ADR-002 | Hard line between code and model: identity, entitlement, dates and money in code; judgment in the model |
| ADR-003 | Four-outcome enum (`APPROVE` / `DENY` / `REFER_SIU` / `ESCALATE`); the co-pilot never finalises |
| ADR-004 | Entitlement checked in code before any agent runs; every call audited, granted or denied |
| ADR-005 | Fraud referral comes from the model reading the file with cited evidence; the fraud score is an input, not the trigger |
| ADR-006 | Coverage must cite retrieved clause IDs; code rejects any citation not in the retrieved set |
| ADR-007 | All data synthetic, from a seeded in-repo generator |

#### Step 8: Build the seed evaluation set
I wrote **20 evaluation cases** covering all four outcomes plus adversarial attacks. Each case records the expected outcome, the evidence a correct answer must cite, and anything that must *not* appear in the response.

| Expected result | Cases |
|---|---|
| `APPROVE` | 8 (includes adversarial and a high-fraud-score-but-clean file) |
| `DENY` | 5 (includes a prompt-injection case) |
| `REFER_SIU` | 3 |
| `ESCALATE` | 3 |
| `REFUSED` (access) | 1 |

**Adversarial families seeded:**

| Family | Cases | Example |
|---|---|---|
| Prompt injection | 2 | A repair estimate containing hidden instructions to the model |
| Cross-record leakage | 1 | A request that tries to pull data from another team's claim |
| Forced auto-decision | 1 | A user demanding the claim be finalised without an adjuster |
| Access refusal | 1 | An unentitled user requesting a claim |

These 20 cases are the starting point for the 150-case suite in Milestone 5.

### Milestone 1 deliverables

| # | Deliverable | File |
|---|---|---|
| 1 | Engagement charter (3 systems + integration points) | [`docs/charter/engagement-charter.md`](docs/charter/engagement-charter.md) |
| | Integration inventory | [`docs/charter/integrations.md`](docs/charter/integrations.md) |
| 2 | Stakeholder map and RACI | [`docs/charter/stakeholders.md`](docs/charter/stakeholders.md) |
| 3 | Four quantified success criteria + guardrails | [`docs/charter/success-criteria.md`](docs/charter/success-criteria.md) |
| 4 | Decision log (7 dated decisions) | [`DECISIONS.md`](DECISIONS.md) |
| 5 | 20 seed evaluation cases | [`evals/m1_seed_cases.jsonl`](evals/m1_seed_cases.jsonl) |
| | Notebook that generates and checks all of the above | [`notebooks/ClaimSense_M1_Discovery.ipynb`](notebooks/ClaimSense_M1_Discovery.ipynb) |

---

## Repository layout

```
claimsense/
├── README.md
├── DECISIONS.md
├── docs/
│   └── charter/
│       ├── engagement-charter.md
│       ├── integrations.md
│       ├── stakeholders.md
│       └── success-criteria.md
├── evals/
│   └── m1_seed_cases.jsonl
└── notebooks/
    └── ClaimSense_M1_Discovery.ipynb
```

Folders for application code, containers, CI and infrastructure will be added from Milestone 2 onward.

## Reproducing Milestone 1

The Milestone 1 notebook uses only the Python standard library. It needs no API keys and no installs.

```bash
git clone <this-repo-url>
cd claimsense
jupyter notebook notebooks/ClaimSense_M1_Discovery.ipynb   # then Run All
```

Every section checks itself and prints `OK - ...` when it passes. The final cell confirms that all five Milestone 1 deliverables exist and meet the brief's requirements. Output is written to `claimsense_m1/`.

## Next: Milestone 2

Turn the discovery findings into a contract, a price and a design:
- a two-page statement of work
- an architecture diagram with the code/model boundary marked
- a pricing model with its assumptions stated
- a five-item risk register
- typed input/output contracts for all three client systems
- synthetic data shapes
- a provisioned AWS account

---

## Data notice

This project uses **synthetic data only**. All claims, policies, documents, claim and policy IDs and users are fabricated, and none relate to any real person or insurer.

## Author

**Anju Mercian**: capstone project for the Forward-Deployed Engineering course.
# ClaimSense-AI
