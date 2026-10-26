# Architecture Decisions & Traceability — EPF-Guide: KWSP Policy & Withdrawal Assistant

**Version:** 1.0
**Author:** Kirthiggan A/L Saravanan (A24AI0042)
**Course:** AI Engineering — Topic 2 Lab

Diagram: `docs/week2_architecture_v1.png`

---

## Step C1 — Simplest architecture

`User Query → Chat Interface → Language Model Service → Response`

This simple flow cannot satisfy the requirements on its own: a model alone has no access to current KWSP documents, cannot cite sources, and tends to describe obsolete schemes. The additions below are each justified by a requirement.

---

## Step C2 — Architecture needs from requirements

| Requirement ID | What does the requirement need? | Architecture Response |
|---|---|---|
| FR1 / FR2 / NFR2 | Answers must match current KWSP rules. | Knowledge access over a curated KWSP document set |
| FR3 / RAI1 | Every document-based answer needs a verifiable source reference. | Source tracking (document name + section/page) carried with retrieved passages |
| FR5 / NFR5 / O2 | Obsolete schemes must not be presented as active. | Knowledge source holds current documents only, with status/date metadata |
| FR4 / RAI3 | The system must abstain when evidence is insufficient. | Grounding & citation check that can trigger an abstain/redirect response |
| FR7 / RAI2 | Out-of-scope requests must be refused with a redirect. | Scope / safety control before generation |
| RAI4 | No sensitive identifiers requested or retained. | Input check for identifiers; privacy-safe logging |
| NFR4 | Document updates without rebuilding the system. | Separate document ingestion/update path |
| NFR1 / NFR3 | Latency and cost within thresholds. | Keep the path short: one model call per answer, lightweight checks |
| Part B (all criteria) | Evidence must be collectable. | Evaluation points on the output and a PII-free interaction log |

---

## Step C4 — Architecture Decision Records

| # | Decision | Requirement / Constraint | Alternatives Considered | Selected Option | Trade-off Accepted |
|---|---|---|---|---|---|
| ADR1 | Include knowledge access over KWSP documents | FR1, FR2, FR3, NFR2; answers must be grounded in official information | Base model only; fine-tune the model on KWSP text; static keyword FAQ; dynamic knowledge access | Dynamic knowledge access over a curated document set | More system complexity and an ingestion/maintenance burden, in exchange for grounding, source traceability and easy updates. |
| ADR2 | Keep only current, active documents in the knowledge source (with status/date metadata) | FR5, NFR5, O2; obsolete schemes must not be presented as active | Ingest all KWSP documents including archived ones; rely on prompt instruction to ignore old schemes | Current-only knowledge source with metadata | Manual curation effort, and the system cannot answer historical questions (it says the scheme is not covered). |
| ADR3 | Add a grounding & citation check after generation, with abstention | RAI1, RAI3, FR4; answers must be supported and the system must say when it cannot answer | Trust the model output; prompt-only instruction to cite | Explicit post-generation check with abstain/redirect | Extra latency and cost (pressure on NFR1/NFR3), and more "cannot answer" replies. |
| ADR4 | Add an explicit scope / safety control before generation | RAI2, FR7; out-of-scope requests must be refused | Prompt instruction only; explicit scope-control component | Explicit scope control | Possible false refusals and additional processing time. |

---

## Initial Requirements Traceability Matrix

| Objective | Requirement | Evaluation Criterion | Architecture Element | Later Evidence |
|---|---|---|---|---|
| O1 | FR2 / FR3 / RAI1 | Source citation and support (100% presence; ≥ 95% match) | Knowledge access + source tracking; grounding & citation check | Citation audit on the evaluation set |
| O1 | FR6 | Multi-category scenario completeness (≥ 80%) | Knowledge access over multiple withdrawal documents + language model service | Multi-scenario test results |
| O2 | FR5 / NFR5 | Obsolete-scheme error rate (≤ 5%) | Current-only knowledge source with metadata; grounding check | Adversarial query test results |
| O2 | NFR4 | Document-update time (< 15 min) | Document ingestion/update path | Controlled update test log |
| O3 | FR1 / FR2 / NFR2 | Correct response rate (≥ 85%) | Knowledge access + language model service | Curated question-set scoring |
| O3 | FR7 / RAI2 | Refusal success (≥ 95%); false refusals (≤ 10%) | Scope / safety control | Scope test-set results |
| O3 | NFR1 / NFR3 | p90 latency (< 5 s); cost per 100 queries | Short processing path; evaluation/logging point | Latency and usage logs |

---

## Final review checklist

- **Does each major component support a requirement or evaluation need?** Yes — see the C2 table and the matrix; no component exists without a requirement.
- **Is any important data source, safety control, or evaluation point missing?** The knowledge source, scope control, grounding check, ingestion path and evaluation/logging points are all present. Multi-turn memory is intentionally left out of V1.
- **Have I added any technology or component not yet justified?** No specific model, vector database, framework or UI toolkit is named; components are labelled by function only.
- **Is there a simpler architecture that could still satisfy the requirements?** Not without dropping FR3, RAI1 or NFR5. The grounding check could be merged into the generation step if latency becomes a problem.

---

## Reflection

**Which architecture decision involved the most important trade-off? Which requirement drove the decision, and what trade-off did you accept?**

ADR3 (grounding & citation check with abstention). It is driven by RAI1 and RAI3, which exist because wrong or unsupported financial information can harm members. The trade-off is that the extra check adds latency and cost, which pulls against NFR1 and NFR3, and it will make the assistant say "I cannot confirm this" more often. I accept that because a slower, more cautious answer is better than a fast answer that is wrong. If measured latency exceeds the 5-second target, I will revisit this decision first.

---

## Git commit

```bash
git add docs/week2_architecture_v1.png docs/week2_architecture_decisions.md
git commit -m "Week 2: Add V1 architecture, ADR, and traceability"
git push
```
