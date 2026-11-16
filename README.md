# EPF-Guide: KWSP Policy & Withdrawal Assistant

> An AI assistant designed to help Malaysian contributors navigate complex EPF (KWSP) account structures and withdrawal eligibility, strictly grounded in official provident fund guidelines.

**Author:** KIRTHIGGAN SARAVANAN
**Course:** SAIA 3113 AI Engineering, Faculty of Artificial Intelligence, Universiti Teknologi Malaysia

## Problem Statement
Malaysian citizens frequently struggle to determine their eligibility for EPF withdrawals because the rules—especially regarding the Persaraan, Sejahtera, and Fleksibel accounts—are highly conditional and buried in fragmented PDFs. EPF-Guide provides a document-grounded retrieval system that instantly evaluates user scenarios against current, active policies.

## Evidence of the Problem
- The recent restructuring of EPF accounts introduced complex new rules that require contributors to cross-reference multiple guidelines to understand their withdrawal limits.
- Official guidelines are separated into distinct PDFs by withdrawal type (Housing, Education, Health), making multi-scenario research tedious.

## Initial Related Work / Existing Approaches
- **Physical EPF Branches** — highly accurate but unscalable and subject to long wait times.
- **Static KWSP Portal** — authoritative but requires manual synthesis of multiple PDFs for complex queries.
- **General-purpose LLMs** — heavily prone to hallucinating outdated pandemic-era withdrawal schemes (e.g., i-Sinar) that are no longer valid.

## Engineering Gap / Limitation
There is no automated, RAG-based assistant that reliably restricts its generation strictly to current, active EPF policies while synthesizing rules across multiple withdrawal categories.

## Project Objectives
- **O1:** To design a document-grounded AI assistant utilizing official KWSP public guidelines.
- **O2:** To develop a retrieval pipeline optimized to prevent hallucination of obsolete financial schemes.
- **O3:** To evaluate the system for factual groundedness, exact-match accuracy, and safety.

## Target Users
Malaysian working adults, young professionals, and retirees managing their EPF contributions.

## Data Sources
- KWSP Account Restructuring Policy Guidelines — https://www.kwsp.gov.my/
- KWSP Withdrawal Terms and Conditions (Housing/Education) — https://www.kwsp.gov.my/

Only URLs and ingestion scripts are kept in this repo. Source documents are not redistributed.

## Repository Structure
```text
├── README.md
├── docs/          # Week 1-3 documents (observation sheet, requirements, architecture, data inventory, pipeline decisions)
├── data/          # data sources and ingestion outputs (no copyrighted files)
├── src/           # application code
├── notebooks/     # week3_pipeline.ipynb (V1 data pipeline) and later experiments
├── evaluation/    # test sets and results
└── prompts/       # prompt templates
```

## Progress Log

| Week | Topic | Artefacts |
|---|---|---|
| 1 | Think like an AI engineer | [Observation sheet](week1_observation_sheet.md), [system map](week1_system_map.jpeg), [group reflections](group_reflections.md) |
| 2 | Plan the AI application | [Requirements](docs/week2_requirements.md), [evaluation criteria](docs/week2_evaluation_criteria.md), [V1 architecture](docs/week2_architecture_v1.png), [architecture decisions](docs/week2_architecture_decisions.md) |
| 3 | Engineer the data | [Pipeline notebook](notebooks/week3_pipeline.ipynb), [data inventory](docs/week3_data_inventory.md), [pipeline diagram](docs/week3_pipeline_diagram.png), [pipeline decisions](docs/week3_pipeline_decisions.md), [lineage manifest](data/week3_manifest.json), [pipeline run manifest](data/pipeline_v1/documents/pipeline_manifest.json) |

### Week 3 — V1 Data Pipeline (PDF pathway)
EPF-Guide only needs official KWSP policy PDFs, so Pathway A (PDF/text) was implemented. The notebook runs in Google Colab: **source → ingest → inspect → prepare (page-aware chunking) → validate output → record lineage**. Every prepared chunk keeps its source file, page number, document status/version, access date and SHA-256 checksum, supporting FR3 (source references), RAI1 (verifiable citations), NFR4 (repeatable updates) and NFR5 (current-only documents).

Source PDFs are not redistributed. See [week3_pipeline_decisions.md](docs/week3_pipeline_decisions.md) for the data-sharing decision, stage contracts and traceability.

