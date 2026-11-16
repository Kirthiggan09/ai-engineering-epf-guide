# Week 3 Data Inventory — EPF-Guide: KWSP Policy & Withdrawal Assistant

**Author:** Kirthiggan A/L Saravanan (A24AI0042)
**Course:** SAIA 3113 AI Engineering — Topic 3 Lab
**Pathway used:** A — PDF documents (the only data type the project needs; Pathways B–F are not relevant)
**Dataset type:** Real project data (public KWSP documents), accessed 2026-11-12. The synthetic dummy pack was **not** used.

> All values come from the Colab run of 2026-11-12 (`week3_inspection.json`, `week3_manifest.json`, `pipeline_manifest.json`, `prepared_text.jsonl`). Where the manifests and the documents themselves disagree, both are stated and the gap is listed as a limitation.

---

## A1 — From Topic 2 requirements to data needs

| Requirement ID | Needed information / feature | Candidate source | Why usable / constraint |
|---|---|---|---|
| **FR1** — Explain Akaun Persaraan / Sejahtera / Fleksibel | Account purpose, allocation and basic rules | S01 *Account Restructuring FAQ (EN)*; S03 *Life Cycle Needs appendix* | Official KWSP material on the restructuring. Constraint: must be the current version; S01 carries a version stamp inside the PDF (see inventory). |
| **FR2 / FR3** — Explain withdrawal eligibility/procedure and cite the source | Withdrawal procedure and application requirements, with **source file + page number** on every chunk | S04 *Akaun Fleksibel withdrawal quick guide (EN)*; S05 *Form 9FA (Akaun Fleksibel withdrawal application)* | Official, public. **Coverage constraint:** these cover Akaun Fleksibel withdrawals only. The Housing / Education / Health withdrawal T&C PDFs named in the Week 2 FR2 are **not** in the V1 corpus (see Section 12 of `week3_pipeline_decisions.md`). |
| **FR1 / RAI1** — Legal basis and verifiable sources | Statutory definitions and wording | S02 *EPF Act 1991 (Act 452), printed from the KWSP website* | Authoritative legal text. Constraint: very large (49 pages) and dominates the corpus (81.5 % of chunks). |
| **NFR5 / RAI1** — Do not present obsolete schemes as active | **Status / version / date** per document, access date, URL | Same PDFs + metadata set in notebook Cell P and carried into the manifests | Supports ADR2 (current-only knowledge source). Constraint: versions are not always printed; where unknown they are recorded as `Not verified`. |
| **NFR4** — Update without rebuilding | Stable source identity (filename + SHA-256) and a re-runnable pipeline | Same PDFs | A changed PDF changes its SHA-256, so lineage verification flags it. |

---

## A3 — Data type and inspection evidence

- **Data type:** unstructured text (PDF). Source format (PDF) and pipeline representation (page-level text → chunks with metadata) are separate decisions.
- **Inspection method:** `pypdf` page-by-page extraction (notebook Cell A3); no OCR.
- **Totals:** 5 PDFs, 68 pages, 197,072 extracted characters.
- **Sources inspected:** all five; most detail below for S01 (FAQ), S02 (Act) and S05 (form).

| Source | Pages loaded | Characters | Empty pages (raw extraction) | Repeated header/footer candidates (Cell A3) | Text sample (first prepared chunk) |
|---|---|---|---|---|---|
| S01 | 11 | 14,165 | none | `Final Version V1.1 (29042024)`; `RESTRICTED` | "Page 1 of 11 … FREQUENTLY ASKED QUESTION (FAQs) ON EPF ACCOUNT RESTRUCTURING … 1. What is EPF account restructuring?" |
| S02 | 49 | 157,184 | none | `11/12/26, 1:48 PM EPF Act 1991: Legal Framework Overview` | "EPF Act 1991 / ACT 452 / EMPLOYEES PROVIDENT FUND ACT 1991 / An Act to provide for the law relating to a scheme of savings …" |
| S03 | 2 | 2,954 | none | none | "APPENDIX 1 / NOTE TO EDITORS: SUMMARY OF EPF ACCOUNT RESTRUCTURING INITIATIVE" |
| S04 | 2 | 1,229 | none | none | "You may follow the same steps for the i-Akaun (Member) Web Portal. Tap 'Fleksibel'. Tap 'Withdraw savings'. After logging in … tap 'Withdrawal' …" |
| S05 | 4 | 21,540 | none | 8 lines: `9FA (AHL)`, `KWSP`, `PENGELUARAN AKAUN FLEKSIBEL`, a website line, a contact-centre line, two "inquiries" lines (Malay/English), `V20261101` | "SENARAI SEMAK DOKUMEN PERMOHONAN / LIST OF SUPPORTING DOCUMENTS FOR AKAUN FLEKSIBEL WITHDRAWAL APPLICATION / KWSP 9FA (AHL) 1/2 …" |

**Observed ingestion limitations**
1. **Reading order (S04):** the extracted quick-guide text appears out of step order (the line about logging in to the i-Akaun app and tapping "Withdrawal" appears after "Tap 'Fleksibel'. Tap 'Withdraw savings'."), which is the "broken reading order" case from the lecture. It matters because FR2 is about step-by-step procedure.
2. **Pages with no body text (S01):** after cleaning, no chunks exist for FAQ pages **6** and **11** (pages 1–5, 7–10 have chunks). Raw extraction reported no empty pages, so these pages most likely contained only stamps/page-number lines that cleaning removed. *Check pages 6 and 11 of the PDF by eye to confirm nothing was lost (e.g. an image-only table).*
3. **Residual footer (S02):** every Act page ends with a browser-print footer `https://www.kwsp.gov.my/en/others/resource-centre/references/epf-act-1991?utm_source=chatgpt.com n/49`. Because the page counter changes, exact-match removal could not remove it; it remains in 49 of the 260 Act chunks (about 3.3 % of Act chunk text).
4. **Bilingual form (S05):** the form is Malay/English (27 of 28 chunks contain Malay words), whereas V1 targets English answers (FR8).
5. **Version stamps removed from text:** `Final Version V1.1 (29042024)` (S01) and `V20261101` (S05) were treated as repeated boilerplate and removed from the chunk text, so the version should have been kept in the metadata (`document_version`); see the inventory table for what was actually recorded.

---

## A4 — Data inventory checkpoint

| Source ID | Filename / URL | Type and purpose | Version / date / owner | Access / rights | Inspection evidence |
|---|---|---|---|---|---|
| **S01** | `Account Restructuring FAQ-EN.pdf` — recorded URL: https://www.kwsp.gov.my/ (homepage only; exact page not recorded) | PDF. Account restructuring FAQ; serves FR1. | Manifest: `Not verified`. The document header states "Final Version V1.1 (29042024)" (not captured in the manifest). Accessed 2026-11-12. Owner: KWSP (EPF). | Public KWSP document; pages carry the marking `RESTRICTED`, and whether it may be shared has not been confirmed. Not redistributed in repo; reuse terms not verified. | 11 pages, 14,165 chars; 22 chunks |
| **S02** | `EPF Act 1991_ Legal Framework Overview.pdf` — https://www.kwsp.gov.my/en/others/resource-centre/references/epf-act-1991 (taken from the page footer printed in the PDF; manifest recorded only the homepage) | PDF printed from a web page. Act 452 text with amendment notes; serves FR1 (legal basis). | Manifest: `Not verified`. Printed 12 Nov 2026 ("11/12/26"). Owner: KWSP (EPF). | Public web page; reuse terms not verified; not redistributed. | 49 pages, 157,184 chars; 260 chunks |
| **S03** | `APENDIX EPF ACCOUNT RESTRUCTURING SET TO ADDRESS MEMBERS LIFE CYCLE NEEDS_25.4.2024 1 (1).pdf` — manifest records only the homepage; a link to this file is listed on KWSP's 25 Apr 2024 announcement page: https://www.kwsp.gov.my/documents/d/guest/apendix-epf-account-restructuring-set-to-address-members-life-cycle-needs_25-4-2024-1-pdf (not independently verified) | PDF. Summary of the restructuring initiative; serves FR1. | `2024-04-25` (taken from the filename). Accessed 2026-11-12. Owner: KWSP. | Same as S01 | 2 pages, 2,954 chars; 6 chunks |
| **S04** | `Panduan Mudah Pengeluaran Akaun Fleksibel_ENG.pdf` — recorded URL: https://www.kwsp.gov.my/ (homepage only; exact page not recorded) | PDF. Quick guide to Akaun Fleksibel withdrawal steps; serves FR2. | `Not verified`. Accessed 2026-11-12. Owner: KWSP. | Same as S01 | 2 pages, 1,229 chars; 3 chunks (reading-order issue) |
| **S05** | `KWSP 9FA (AHL)_BORANG PERMOHONAN PENGELUARAN AKAUN FLEKSIBEL_25042024.pdf` — recorded URL: https://www.kwsp.gov.my/ (homepage only; exact page not recorded) | PDF form. Application requirements and supporting documents; serves FR2. | Manifest: `2024-04-25` (taken from the filename). The PDF footer repeats `V20261101`, which looks like a later version code; which version was downloaded is not confirmed. Owner: KWSP. | Same as S01 | 4 pages, 21,540 chars; 28 chunks |

**Raw vs prepared storage**
- **Raw PDFs:** kept only in the temporary Colab runtime and on your own machine. Not committed.
- **Prepared data:** `data/processed/prepared_text.jsonl` (Part B) and `data/pipeline_v1/documents/prepared_text.jsonl` (Part C). The two files are byte-identical (same SHA-256). See the rights note in `week3_pipeline_decisions.md` before committing them.

**Prepared fields kept with every chunk:** `chunk_id`, `text`, `source`, `source_title`, `source_url`, `source_sha256`, `accessed_date`, `document_version`, `doc_status`, `page`, `processing`, `status`.

---

**Checkpoint A — can I explain which requirement needs this data and show a loaded sample?**
Yes. FR1 is served by S01, S03 and S02; FR2/FR3 by S04 and S05 (Akaun Fleksibel withdrawal only), with page numbers kept on all 319 chunks for RAI1. NFR5 is supported by the per-document `doc_status`, version and access date. The text samples above come from the real run.
