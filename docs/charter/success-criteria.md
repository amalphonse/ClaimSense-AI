# Success criteria

Format: `[metric] [threshold] [population]`. Thresholds are *proposed* and must be agreed with the Head of Claims before M2.

| # | Dimension | Criterion | How measured | Whose need | Baseline |
|---|---|---|---|---|---|
| 1 | Decision accuracy | First decision matches the gold-labelled outcome on >= 85% of cases in the held-out eval suite (20 cases at M1, 150 at M5), and >= 75% on every one of the four outcome classes | eval runner: exact match of outcome enum vs gold label; per-class accuracy reported | Head of Claims; Claims QA Lead | TBD: measure adjuster-vs-adjuster agreement on the same 20 cases in discovery |
| 2 | Pipeline latency | End-to-end p95 latency <= 120 seconds per claim, and p50 <= 60 seconds, measured on the eval suite against the deployed AWS stack | request timestamps in CloudWatch; p50/p95 on dashboard (M5) | Head of Claims ("minutes, not hours") | TBD: current manual first pass is "hours", so time-study 10 claims |
| 3 | Adjuster confirmation rate | At least 80% of co-pilot first decisions are confirmed by the licensed adjuster without changing the outcome, on all pilot claims reviewed in week 5 | confirm / override recorded per claim in the audit log; rate = confirmed / reviewed | Licensed adjusters; Claims QA Lead | n/a (new measure); overrides must carry a reason code |
| 4 | Fraud routing accuracy | REFER_SIU recall >= 90% on fraud-labelled cases, with false SIU referrals <= 10% of non-fraud cases, on the eval suite | confusion matrix for the REFER_SIU class vs gold label | SIU Lead; Head of Claims | TBD: get current threshold-based referral and hit rates from SIU |

## Guardrails (do-no-harm, zero tolerance)

- Claim data returned to an unentitled user = 0 across all access-test cases in the eval suite, with 100% of refusals present in the audit trail
- Decisions finalised without a licensed adjuster = 0 across all claims, including forced auto-decision cases
- Coverage citations not present in the retrieved clause set = 0% of coverage outputs on the eval suite
