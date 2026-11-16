# Week 3 Pipeline Decisions, Stage Contracts & Traceability — EPF-Guide

**Author:** Kirthiggan A/L Saravanan (A24AI0042)
**Course:** SAIA 3113 AI Engineering — Topic 3 Lab
**Pipeline version:** `week3-v1` | **Pathway:** A (PDF documents) | **Notebook:** `notebooks/week3_pipeline.ipynb`
Diagram: `docs/week3_pipeline_diagram.png` | Inventory: `docs/week3_data_inventory.md`

> All values below come from the real Colab run on 2026-11-12.

---

## 1. Topic 2 requirements this pipeline serves

| Requirement | Why the data pipeline matters | Later evaluation criterion it affects |
|---|---|---|
| **FR1** | Account-structure rules must be in the knowledge source (S01, S03, S02) | Correct response rate (NFR2 ≥ 85 %) |
| **FR2 / FR3** | Withdrawal procedure/requirements (S04, S05) must be chunked with **source file + page** | Citation presence/match (RAI1) |
| **NFR5 / FR5** | Only current documents are prepared; each chunk carries `doc_status`, version and access date | Obsolete-scheme error rate (≤ 5 %) |
| **RAI1** | Every chunk traces to a document, page and SHA-256 checksum | Citation-source match (≥ 95 %) |
| **NFR4** | Replacing a PDF and re-running regenerates chunks and lineage; checksums reveal changes | Document-update time (< 15 min) |

The pipeline implements the **Document Ingestion / Update** and **Policy Knowledge Source** parts of the Week 2 V1 architecture (ADR1, ADR2). It creates no embeddings or vector index (later topics).

## 2. Selected sources

Five public KWSP documents, accessed 2026-11-12, all with `doc_status = "current"` (68 pages, 197,072 extracted characters):

| ID | Document | Serves |
|---|---|---|
| S01 | Account Restructuring FAQ (English) | FR1 |
| S02 | EPF Act 1991 (Act 452), printed from the KWSP website | FR1 (legal basis) |
| S03 | Life Cycle Needs appendix (2024-04-25) | FR1 |
| S04 | Akaun Fleksibel withdrawal quick guide (English) | FR2 |
| S05 | Form 9FA — Akaun Fleksibel withdrawal application | FR2 |

Full inventory, exact URLs and inspection evidence: `docs/week3_data_inventory.md`.

## 3. Part B comparison and evidence

Same five sources, same chunk size (800 characters), same cleaning, two approaches (Cell 5A1 output):

| Measure | A — fixed-size (whole document) | B — page-aware paragraph/line-aware |
|---|---|---|
| Chunks | 243 | 319 |
| Length min / mean / max (chars) | 27 / 792.7 / 800 | 20 / 601.4 / 800 |
| Chunks with a page number | **0.0 %** | **100.0 %** |
| Start mid-sentence (first character lowercase) | 72.4 % | **21.0 %** |
| End without terminal punctuation | 96.3 % | 71.2 % |
| `needs_review` chunks | 1 | 6 |

| Approach A | Approach B | Observed evidence | Selected approach and trade-off |
|---|---|---|---|
| Fixed 800-character slices over the concatenated document | One page at a time; whole paragraphs, then lines, then words, up to 800 characters | B keeps a page number on all 319 chunks (A has none) and starts far fewer chunks mid-sentence (21.0 % vs 72.4 %) | **Selected: B (page-aware).** *Benefit:* every chunk can be cited by document and page, which FR3 and RAI1 require; chunks are less likely to start in the middle of a sentence or condition. *Accepted limitations:* 76 more chunks (+31 %) and shorter chunks on average (601 vs 793 chars); 6 chunks flagged `needs_review`; 71.2 % of chunks still end without terminal punctuation (headings, bullet lists, form fields); a condition that continues across a page boundary is split; no overlap in V1. |

The mid-sentence and end-punctuation figures are simple proxies, not a quality assessment (Topic 4).

**The six `needs_review` chunks** are all very short amendment notes from the Act (e.g. `[Subs. Act A1611/2019]`, `[Am. Act A1080/2000]`), under 40 characters. They are kept, not deleted, because amendment history may matter for FR1.

**Cleaning decisions**
- Always: collapse repeated spaces/tabs and 3+ blank lines (formatting noise only).
- **Repeated-line removal: ON.** Justified by the Cell A3 candidates, which were all clearly boilerplate: stamps/headers in S01 (`Final Version V1.1 (29042024)`, `RESTRICTED`), a browser-print header in S02, and the running title, contact lines and form code in S05.
- **Side effects observed after cleaning:**
  1. Version stamps (`V1.1 (29042024)`, `V20261101`) and the `RESTRICTED` marking no longer appear in the chunk text, so the version has to be kept in the metadata (`document_version`). In this run S01 was recorded as `Not verified` although its header states V1.1 (29042024), and S05 as 2024-04-25 (from the filename) although its footer shows `V20261101`. This is a provenance gap in the manifests, listed in Section 12.
  2. FAQ pages 6 and 11 produced no chunks. Likely cause: only stamps and a page-number line remained after cleaning, so the page fell under the 20-character minimum and was skipped. To be confirmed by viewing those pages.
  3. The Act's page footer (`…epf-act-1991?utm_source=chatgpt.com n/49`) was **not** removed because its page counter changes; it remains in 49 of 260 Act chunks (about 3.3 % of Act chunk text). Exact-match removal cannot catch such lines; normalising digits before matching is a later improvement.

## 4. Chosen preparation configuration

| Setting | Value | Reason |
|---|---|---|
| Method | page-aware | Page metadata for FR3 / RAI1 (evidence in Section 3) |
| Chunk size | 800 characters | Keeps a condition and its qualifier together while staying topic-focused (lecture: too small loses context, too large mixes topics) |
| Overlap | none | Kept simple for V1; revisit when retrieval is evaluated |
| Repeated-line removal | True | Candidates were boilerplate (Section 3); side effects recorded |
| Minimum page / chunk size | 20 / 40 characters | Flags empty pages and tiny chunks (`status = needs_review`) |
| Documents included | `doc_status = "current"` only (all five) | ADR2 / NFR5 |

Part C (Cell 8A) reads the same configuration and calls the same helper functions as Part B. **Consistency check:** the Part B and Part C `prepared_text.jsonl` files are byte-identical (same SHA-256), matching "Identical records in both parts: True".

## 5. ETL / ELT decision

**Decision: hybrid (raw landing + staged transformations).** The original PDFs stay unchanged under `data/raw/`; the pipeline then inspects, cleans and chunks them in stages and stores the prepared output and lineage.
- Not plain ETL: the raw PDFs must be retained so chunking can be re-run if retrieval evaluation shows a better strategy is needed. This run already shows why (Section 3: footer lines and removed version stamps).
- Not plain ELT: there is no warehouse, and PDF text must be extracted and cleaned before it is usable.
- Supports NFR4 (repeatable update) and RAI1 (every output traces to an unchanged source).

## 6. Batch vs streaming

**Batch.** KWSP policy documents change infrequently and only when KWSP publishes a new version, so there is no real-time feed. The pipeline runs on demand when a document is added or replaced; streaming would add complexity with no benefit, and the NFR4 target (< 15 minutes) is a batch re-run.

## 7. Output locations and results

| Artefact | Location |
|---|---|
| Part B prepared chunks | `data/processed/prepared_text.jsonl` |
| Part B lineage manifest | `data/week3_manifest.json` |
| Inspection summary | `data/week3_inspection.json` |
| Part C outputs + run manifest | `data/pipeline_v1/documents/prepared_text.jsonl`, `data/pipeline_v1/documents/pipeline_manifest.json` |
| Shareable evidence preview | `data/evidence/prepared_text_preview.json` |

**Results:** 319 chunks, all `doc_status = current`, all with a page number (S01: 22, S02: 260, S03: 6, S04: 3, S05: 28); 313 `ready`, 6 `needs_review`. The Part B manifest lists five source-to-output entries (S01–S05) mapping to one prepared file.
**Repeatability (Cell 9):** run in the notebook after Cell 8A; its output is stored in the executed notebook. The Part C manifest (`pipeline_manifest.json`) was written after the Part B manifest, consistent with that rerun.

## 8. Provenance and access caveats

- **Exact URLs:** the manifests record only `https://www.kwsp.gov.my/` for all five sources. S02's exact page is known from the footer printed in its PDF (`https://www.kwsp.gov.my/en/others/resource-centre/references/epf-act-1991`); for the other four sources the exact download page was not recorded. This is a known provenance limitation.
- **Versions:** the manifests record `Not verified` for S01, S02 and S04, `2024-04-25` for S03, and `2024-04-25` for S05 (from its filename). The PDFs themselves show S01 = `V1.1 (29042024)` and S05 = `V20261101`, so the recorded S01/S05 versions are incomplete or possibly out of date.
- **Marking:** every page of S01 carries the word `RESTRICTED`; it has not been confirmed that this document may be shared.
- `access_rights` is recorded as *public KWSP document; reuse terms not verified*. A SHA-256 checksum proves a file did not change; it does not prove permission to share or data quality.
- The README says source documents are **not redistributed**, but the prepared JSONL contains KWSP text. **Decision: the full prepared text is not committed** (reuse terms are unverified and S01 carries a `RESTRICTED` marking). `prepared_text.jsonl` is stored on the author's own computer (downloaded from Colab); the repo holds the manifests, the inspection summary and `data/evidence/prepared_text_preview.json` (counts and truncated 120-character previews) as non-sensitive execution evidence. It can be added later if KWSP's terms of use allow it.
- The documents are public, so no personal data is involved. No API key is used.
- Quality, fitness-for-purpose and Responsible AI assessment of the data are Topic 4 work.

## 9. Stage contracts and failure responses (as implemented)

| Stage | Contract / evidence | Failure response |
|---|---|---|
| **Ingest** | Each chosen PDF exists under `data/raw`, is explicitly named, is `doc_status = current` and has a real access date | Stop for a missing file or placeholder date; non-current documents are excluded and listed |
| **Inspect** | PDF is readable; pages, characters per page, empty pages, text sample and repeated-line candidates are printed | Stop if a PDF has no extractable text (OCR needed); flag and skip empty pages |
| **Prepare** | Chosen method yields non-empty chunks, each with `source`, `page`, `doc_status` | Stop if no chunks; chunks under 40 characters get `needs_review` |
| **Validate** | Output exists, is non-empty, every source contributed chunks, every chunk has a page number and `current` status | Stop for missing/empty output or a failed assertion |
| **Record lineage** | Each source SHA-256 maps to the output SHA-256 in `week3_manifest.json` and `pipeline_manifest.json`; a rerun reproduces identical fingerprints | Stop and report a checksum mismatch, missing mapping or changed output |

In this run no stage stopped the pipeline. Two quality flags were raised instead (Section 3): pages that produced no chunks (S01 pages 6 and 11) and residual footer lines (S02). The diagram annotates failure point 1 (empty or image-only page at Inspect; the variant seen in this run was boilerplate-only pages S01 p.6 and p.11 skipped after cleaning) and failure point 2 (missing/empty output or checksum mismatch at Validate / Record lineage).

## 10. Documented adaptations of the lab template

1. **Page-aware chunking** replaces the template's whole-document paragraph splitter, which sets `page = None`; FR3/RAI1 need page numbers.
2. **Multiple PDFs:** Cell 7 `execute_pipeline` accepts a list of explicitly selected files when `kind == "documents"`; other kinds behave as in the template.
3. **Lineage enrichment:** Cell 6A also stores title, URL, access date, document version and `doc_status` per source.
4. **Shared helpers:** Part B and Part C use the same functions and configuration so they cannot diverge.

## 11. Updated traceability (extends the Week 2 matrix)

| Objective | Requirement | Evaluation criterion | Architecture element | Week 3 evidence | Coverage |
|---|---|---|---|---|---|
| O1 | FR3 / RAI1 | Citation presence 100 %, match ≥ 95 % | Document Ingestion → Policy Knowledge Source | 319 / 319 chunks have `source` + `page`; `week3_manifest.json` | Supported by the data |
| O1 | FR1 | Correct response rate ≥ 85 % | Knowledge access | S01, S02, S03 prepared (28 + 260 chunks) | Covered |
| O1 | FR2 | Correct response rate ≥ 85 % | Knowledge access | S04, S05 prepared (31 chunks) | **Partial** — Akaun Fleksibel only; Housing/Education/Health T&Cs not yet ingested |
| O2 | FR5 / NFR5 | Obsolete-scheme error rate ≤ 5 % | Current-only knowledge source with metadata | `doc_status`, version and access date on every chunk | Supported; S01 and S05 version stamps not captured in the manifests |
| O2 | NFR4 | Document-update time < 15 min | Ingestion / Update path | Part B / Part C outputs identical; checksums in manifests | Supported |
| O3 | FR2 / NFR2 | Correct response rate ≥ 85 % | Knowledge access (chunk quality) | Part B comparison (Section 3) | Supported |

## 12. Known limitations and left for later topics

- **Scope gap against FR2:** the Week 2 requirement mentions Housing, Education and Health withdrawals, but the V1 corpus holds Akaun Fleksibel withdrawal documents. Accepted for V1; the next iteration should add the missing withdrawal PDFs (re-running this pipeline is the update path) or narrow the FR2 wording.
- **Corpus imbalance:** the Act is 81.5 % of all chunks (260 of 319), while the quick guide has 3. Retrieval for practical questions may be dominated by legal text; consider metadata filters or per-source weighting in the retrieval topic.
- **Reading order and tables:** S04's step order looks scrambled; table-aware or layout-aware extraction is needed for procedure documents.
- **Boilerplate:** page-numbered footers (S02) need digit-normalised matching.
- **Language:** S05 is bilingual Malay/English; V1 targets English.
- **Also later:** embeddings and vector index; overlap and size tuning against retrieval results; OCR for scanned pages; full data-quality and Responsible AI data review (Topic 4).

## 13. Improvements identified but not applied to this run

The results in this document come from the first run, which was **not** repeated. Review of that run suggested these improvements for the next iteration:

| Improvement | Reason |
|---|---|
| Record the exact download page and the version stamp seen in each PDF (e.g. S01 `V1.1 (29042024)`, S05 `V20261101`) in Cell P before running | Closes the provenance gaps in Section 8 |
| Print version/stamp candidates in Cell A3 before repeated-line removal | The stamps were removed from the text before being recorded |
| Warn about pages that end up without chunks and store them in `pipeline_manifest.json` | FAQ pages 6 and 11 were skipped without a visible warning |
| Normalise page counters (`n/49`, `Page n of 11`) before matching repeated lines | Removes the residual Act footer (about 3.3 % of Act chunk text) |
| Add the Housing / Education / Health withdrawal PDFs | Closes the FR2 coverage gap (Section 12) |

Applying any of these changes chunk metadata or counts, so the evidence in this document would need to be regenerated.
