# Requirements Specification — EPF-Guide: KWSP Policy & Withdrawal Assistant

**Version:** 1.0
**Author:** Kirthiggan A/L Saravanan (A24AI0042)
**Course:** AI Engineering — Topic 2 Lab

---

## Section 0 — Project Framing & Application Scope Checkpoint

| Element | Content |
|---|---|
| **Problem Evidence** | (1) KWSP restructured member accounts from 11 May 2024: new contributions are split 75% Akaun Persaraan / 15% Akaun Sejahtera / 10% Akaun Fleksibel, and withdrawal rules differ by account and by purpose. Source: KWSP Account Restructuring page, https://www.kwsp.gov.my/account-restructuring *(add access date)*. (2) KWSP withdrawal guidance is published as separate documents per withdrawal type (Housing, Education, Health), so a member with a multi-purpose scenario must read several PDFs and combine the conditions manually. Source: KWSP withdrawal T&C documents *(add exact document titles/URLs from your /data/ list)*. |
| **Existing Approaches** | **(a) Physical EPF branches** — highly accurate, personalised, but not scalable and subject to wait times. **(b) KWSP portal / i-Akaun** — authoritative and current, but information is split across pages and PDFs; the member must synthesise rules for complex cases. **(c) General-purpose LLM chatbots** — fast and conversational, but not restricted to current KWSP documents and prone to presenting obsolete pandemic-era schemes (e.g. i-Sinar) as if still available. |
| **Project / Engineering Gap** | No assistant is restricted to current, active KWSP policy documents while also (i) synthesising conditions across several withdrawal categories, (ii) citing the exact source for each claim, and (iii) refusing to present obsolete schemes as active. This is a practical/engineering gap (grounding, traceability, currency), not a claim of scientific novelty. |
| **Objective O1** | To design a document-grounded AI assistant that answers EPF account and withdrawal questions using official KWSP public guidelines, with source references. |
| **Objective O2** | To develop a retrieval pipeline over a curated, current-only document set that prevents the assistant from presenting obsolete or unsupported schemes as active. |
| **Objective O3** | To evaluate the system for factual correctness / groundedness, safe handling of out-of-scope requests, and responsiveness against defined thresholds. |
| **Intended Users** | Malaysian working adults, young professionals, and retirees who manage their own EPF (KWSP) contributions. |
| **Supported Tasks** | (1) Explain the purpose, allocation and basic rules of Akaun Persaraan, Sejahtera and Fleksibel. (2) Explain withdrawal eligibility conditions for Housing, Education and Health withdrawals. (3) Work through a user-described scenario against the documented conditions, including scenarios that touch more than one withdrawal category. (4) Point the user to the source document/section and to the official KWSP channel for the next step. |
| **Information / Knowledge Domain** | Official, publicly available KWSP policy documents on account structure and the Housing / Education / Health withdrawal categories (current versions only). |
| **Operating Context** | Web chat interface used by individuals on laptop or phone for self-service information before contacting KWSP; informational use only, not a transaction channel. |
| **Important Constraints** | Source PDFs are not redistributed in the repo (only URLs and ingestion scripts); limited corpus (2–3+ PDFs for V1); student project budget and time; no access to member accounts or personal data; policies can change, so documents must be re-ingested when KWSP updates them. |
| **Explicit Out-of-Scope Tasks** | The system must not: approve, reject or guarantee any withdrawal application; look up balances or access member accounts; submit applications; give investment, tax, legal or personal financial planning advice; predict dividends; answer about schemes not found in the current document set as if they were active. |
| **Evaluation Focus** | Does the system give correct, source-supported answers on current EPF rules, avoid presenting obsolete schemes as active, refuse out-of-scope requests, and respond fast enough for interactive use? |

---

## Section 1 — Functional Requirements

| # | Requirement — The system SHALL | Scope Boundary — SHALL NOT (where relevant) |
|---|---|---|
| **FR1** | Answer questions about the purpose, contribution allocation and basic rules of Akaun Persaraan, Akaun Sejahtera and Akaun Fleksibel using the current KWSP documents. | Invent account rules, percentages or amounts that are not supported by the source documents. |
| **FR2** | Explain the eligibility conditions for Housing, Education and Health withdrawals, and evaluate a user-described scenario against those documented conditions. | State or imply that an application will be approved or rejected; that decision belongs to KWSP. |
| **FR3** | Provide a source reference (document name and section/page where available) for each answer based on KWSP documents. | Present unsupported information as if it came from an official source. |
| **FR4** | State clearly when the available documents do not contain enough information to answer, and direct the user to the official KWSP channel. | Guess or fill gaps from general model knowledge. |
| **FR5** | Handle queries about schemes or rules that are not in the current document set (e.g. pandemic-era i-Sinar) by stating that they are not covered by the current documents. | Describe such schemes as currently available. |
| **FR6** | Combine conditions from more than one withdrawal category when a user scenario involves more than one, attributing each condition to its source. | Merge conditions from different documents without attribution. |
| **FR7** | Decline requests outside the defined EPF information scope (investment, tax, legal, balance lookup, application submission) and provide an appropriate redirect. | Provide financial-planning, tax or legal advice. |
| **FR8** | Present answers in clear English suitable for general members, keeping official Malay terms (e.g. Akaun Persaraan) where they appear in sources. | — |

---

## Section 2 — Non-Functional Requirements

| # | Quality Attribute | Requirement | Measurement Method | Metric | Threshold |
|---|---|---|---|---|---|
| **NFR1** | Performance | The system shall respond within an interactive latency target. | Measure end-to-end latency over a fixed workload of ≥ 50 test queries. | p90 latency | < 5,000 ms |
| **NFR2** | Correctness | Answers shall be factually consistent with the current KWSP documents. | Score a curated set of ≥ 50 supported questions against reference answers (exact-match on key facts, rubric for the rest). | Correct response rate (%) | ≥ 85% |
| **NFR3** | Cost | Inference cost shall stay within the project budget. | Record model/API token usage over the evaluation workload. | Cost per 100 queries | < US$0.50 (illustrative) |
| **NFR4** | Maintainability | A KWSP document update shall be incorporated without rebuilding unrelated components. | Run a controlled document-update test (replace one document and re-ingest). | Update time | < 15 min |
| **NFR5** | Correctness (currency) | The system shall not present obsolete or non-current schemes as active. | Run an adversarial set of ≥ 20 queries about obsolete/non-existent schemes. | Obsolete-scheme error rate (%) | ≤ 5% |
| **NFR6** | Reliability | The system shall return a valid response for normal queries. | Run the 50-query workload and count failed/empty/error responses. | Failed-response rate (%) | ≤ 2% |

---

## Section 3 — Responsible AI Requirements

| # | Responsible AI Risk | Requirement | Measurement Method | Metric | Threshold |
|---|---|---|---|---|---|
| **RAI1** | Transparency / Grounding (misinformation) | The system shall give a verifiable source reference for document-grounded answers. | Inspect document-grounded responses for citation presence and check whether the cited passage supports the claim. | Citation presence rate; citation-source match rate | 100% presence; ≥ 95% match |
| **RAI2** | Safety / Scope (harmful financial advice) | The system shall refuse out-of-scope queries and redirect, without wrongly refusing in-scope ones. | Run a scope test set of ≥ 20 out-of-scope and ≥ 20 in-scope queries. | Refusal-and-redirect success rate; false-refusal rate | ≥ 95% success; ≤ 10% false refusals |
| **RAI3** | Over-reliance / uncertainty | The system shall abstain and signal uncertainty when evidence is insufficient, and remind users that KWSP makes final decisions. | Run ≥ 15 unanswerable queries and check each reply for abstention plus the reminder. | Appropriate-abstention rate | ≥ 90% |
| **RAI4** | Privacy | The system shall not request or retain sensitive personal identifiers (NRIC, member number, bank details). | Run privacy test cases containing fake identifiers; inspect prompts sent to the model API and stored logs. | Identifiers detected in outgoing payloads or logs | 0 |
| **RAI5** | Fairness / Accessibility | Performance shall not differ beyond tolerance across user types and query styles. | Evaluate a stratified test set (young professional, mid-career, retiree scenarios; formal vs informal / Malay-mixed phrasing) and compare correctness. | Absolute correctness-rate gap between groups | ≤ 10 percentage points |

---

## Reflection

**Which Responsible AI requirement in your project will be the most difficult to measure or enforce technically? Why?**

RAI1 (citation–source match) will be the hardest. Checking that a citation is *present* is easy to automate, but checking that the cited passage actually *supports* the specific claim needs a judgement about meaning, usually by a human reviewer or a second model. EPF rules are full of conditions (age, amount, purpose), and an answer can quote the right document while misstating one condition. That makes the match rate slow to measure at scale and easy to overestimate with a lenient reviewer.

---

## Git commit

```bash
git add docs/week2_requirements.md
git commit -m 'Week 2: Requirements specification (FR, NFR, RAI)'
git push
```
