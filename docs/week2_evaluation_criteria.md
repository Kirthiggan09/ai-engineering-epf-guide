# Evaluation Criteria — EPF-Guide: KWSP Policy & Withdrawal Assistant

**Version:** 1.0
**Author:** Kirthiggan A/L Saravanan (A24AI0042)
**Course:** AI Engineering — Topic 2 Lab

Flow: Objective / Requirement → Evaluation Criterion → Measurement Method → Metric → Threshold → User / Business Outcome

| Objective / Requirement ID | Evaluation Criterion | Capability Bucket | Measurement Method | Metric | Threshold | User / Business Outcome |
|---|---|---|---|---|---|---|
| O3 / FR1 / FR2 / NFR2 | Answer correctness on supported EPF questions | Domain-specific capability | Run a curated set of ≥ 50 supported questions (account rules, Housing/Education/Health conditions) and compare to reference answers; exact-match on key facts (ages, percentages, amounts), rubric for the rest. | Correct response rate | ≥ 85% | Members get reliable answers for routine EPF enquiries before deciding whether to apply. |
| O2 / FR5 / NFR5 | Resistance to obsolete or non-current schemes | Domain-specific capability | Run ≥ 20 adversarial queries about schemes not in the current documents (e.g. i-Sinar) and check whether any reply presents them as active. | Obsolete-scheme error rate | ≤ 5% | Members are not misled into planning around a scheme that no longer exists. |
| O1 / FR3 / RAI1 | Source citation and support | Instruction-following capability | Inspect document-grounded responses for citation presence, then check whether the cited passage supports the answer. | Citation presence rate; citation-source match rate | 100% presence; ≥ 95% match | Members can verify every answer against an official KWSP document. |
| O3 / FR7 / RAI2 | Out-of-scope refusal with low over-refusal | Instruction-following capability | Test with ≥ 20 out-of-scope and ≥ 20 in-scope queries; check refusal-and-redirect and wrongly refused in-scope queries. | Refusal-and-redirect success rate; false-refusal rate | ≥ 95%; ≤ 10% | Reduces the chance of the system giving investment/tax/legal advice while staying useful for real EPF questions. |
| O1 / FR6 | Multi-category scenario synthesis | Generation capability | Run ≥ 15 scenarios that involve more than one withdrawal category; mark each as complete if all relevant conditions are covered and attributed to the right source. | Completeness rate (rubric: complete / partial / wrong) | ≥ 80% complete | Members with multi-purpose scenarios get a single combined answer instead of reading several PDFs. |
| O3 / NFR1 | Response latency | Cost and latency | Measure end-to-end response time over a fixed workload of ≥ 50 queries. | p90 latency | < 5 s | Members receive responses fast enough for interactive use. |
| O3 / NFR3 | Inference cost | Cost and latency | Record token/API usage over the same workload. | Cost per 100 queries | < US$0.50 (illustrative) | Keeps the project affordable to run and demonstrate. |

> **Note:** Thresholds are initial targets set for V1. They can be revised after the first baseline run, with the reason recorded.

---

## Reflection

**Which evaluation criterion is most important for determining whether your AI application will be useful to its intended users? Why?**

Resistance to obsolete or non-current schemes (O2 / NFR5), together with answer correctness. The whole reason for EPF-Guide is the gap left by general-purpose chatbots that confidently describe schemes like i-Sinar that no longer apply. If a member acts on wrong or outdated withdrawal information, the cost is real money and a wasted application, so this criterion decides whether the assistant is trustworthy at all. Latency and cost matter, but a fast, cheap answer that is wrong is worse than a slower correct one.

---

## Git commit

```bash
git add docs/week2_evaluation_criteria.md
git commit -m 'Week 2: Define evaluation criteria'
git push
```
