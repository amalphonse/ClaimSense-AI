# ClaimSense AI: Engagement Charter

*Version 0.1, 2026-11-16. Milestone 1, discovery and scoping.*

## 1. Client and engagement
A motor insurer has engaged us for a five-week forward-deployed engagement, from discovery to
handover. The deliverable is a **claims adjudication co-pilot**. Given a claim, it returns a
first decision, the reasoning behind that decision, and the evidence the reasoning rests on.
A licensed adjuster confirms every outcome.

## 2. Problem statement
Adjusters complete every first pass by hand. One adjuster gathers the claim file, checks
coverage, assesses fraud, total-loss and injury exposure, and writes the first decision. This
takes hours per claim. The client needs it done in minutes.

The mechanism, not the symptom:
- **Integration friction.** The evidence for one claim is spread across three systems with no
  single view, so most of the first pass is spent gathering rather than judging.
- **Unrecorded reasoning.** Decisions vary between adjusters and the reasoning is not written
  down, so outcomes are inconsistent and cannot be audited or defended.
- **Threshold-driven fraud referral.** Referral to the Special Investigations Unit (SIU) fires on
  a score threshold rather than on a reading of the file, so narrative inconsistencies are missed
  and clean claims are over-referred.

Extending the existing rule table does not fix this. Coverage depends on the *wording* of a
policy clause, and fraud depends on the *consistency of a narrative* across documents. Both are
reading tasks.

## 3. Client systems and integration points
| System | What we need from it | Integration point (proposed) | Access |
|---|---|---|---|
| **Claims management** | claim record, loss narrative, claimant, status, assigned adjuster | REST read of claim by ID; write-back of draft decision (idempotent, keyed on claim ID + version) | read + one guarded write |
| **Policy administration** | policy record, in-force dates, cover type, limits, excess, clause IDs | REST read of policy by number; clause wording exported into a searchable policy-clause corpus | read only |
| **Document store** | photos, repair estimates, police reports, medical notes, statements | list and fetch documents by claim ID; text extracted for the model | read only |

All three are reached **only** through a participant-owned data-access service with
authentication. Access control is checked in code before any agent is invoked.

## 4. Fixed constraints (not subject to design)
1. **Deterministic operations are code.** Identity, access control, policy dates and monetary
   calculations are never delegated to a model.
2. **Judgment is delegated to a model and is cited.** Every judgment states the evidence it relied on.
3. **Synthetic data only.** No production or personally identifiable data at any stage.

## 5. Scope
**In scope:** first-pass recommendation in one of four outcomes (APPROVE, DENY, REFER_SIU,
ESCALATE). Cited coverage reasoning from retrieved clauses. Entitlement refusal with an audit
record. Evaluation suite and CI gate. AWS deployment, dashboard, runbook, handover.

**Out of scope:** final settlement or payment, which stays with the licensed adjuster.
Production data. Changes to the client systems themselves. Replacing the SIU investigation.

## 6. Success
Measured by the four quantified criteria and two guardrails in `success-criteria.md`.

## 7. Related artefacts
`stakeholders.md` (stakeholders and RACI) · `success-criteria.md` · `integrations.md` (integration
inventory) · `../../DECISIONS.md` · `../../evals/m1_seed_cases.jsonl`
