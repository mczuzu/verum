# Verum — HTA Evidence Intelligence Platform

Concept architecture · March 2026 · Portfolio artifact

**Prototype:** https://mczuzu.github.io/verum/
Interactive demo · Pembrolizumab NSCLC · Real HTA data · Full four-step workflow

---

## What It Is

Verum mines published HTA (Health Technology Assessment) decision documents across EU agencies — Spanish IPTs (Informes de Posicionamiento Terapéutico), NICE (National Institute for Health and Care Excellence) STAs (Single Technology Appraisals), G-BA (Gemeinsamer Bundesausschuss) dossiers, HAS decisions — extracts the implicit PICO (Population, Intervention, Comparator, Outcome) specifications each agency actually accepted, and maps them cross-country to show where requirements converge and where they diverge.

The result: before you lock a Phase III protocol, you know which comparator satisfies all target markets, which biomarker thresholds need dual assays, and which endpoints must be co-powered.

**Status:** Concept validated. Data model complete. UI prototype live. No paying customers yet.

---

## The Core Insight

Every published HTA decision contains an implicit study specification: the population, comparator, endpoints, and study design that the agency actually accepted. That specification is machine-extractable from narrative PDFs using structured LLM (Large Language Model) extraction.

An IPT is not just a decision document — it is a reverse-engineered study protocol. Mine enough of them and you have a rules engine. The corpus is entirely public. Nobody has structured it into a queryable cross-country database.

---

## The Problem It Solves

**550+ AEMPS IPTs published. 2,500+ EU HTA decisions publicly available. 0 structured as a queryable database.**

Pharma companies run the trials. They have the data. The problem is that AEMPS, NICE, G-BA, and HAS each ask a slightly different question about that same evidence — and each question changes depending on what they've accepted for similar drugs before. Nobody has structured that institutional memory.

HEOR (Health Economics and Outcomes Research) teams face three recurring, avoidable failures:

**01 — Wrong comparator, wrong country**
G-BA uses platinum-pemetrexed as SoC (Standard of Care) for non-squamous NSCLC (Non-Small Cell Lung Cancer). AEMPS and NICE accept platinum-based chemotherapy broadly. A trial designed against one SoC fails AMNOG (Arzneimittelmarktneuordnungsgesetz) assessment regardless of OS (Overall Survival) results.

**02 — Biomarker fragmentation**
AEMPS accepted PD-L1 CPS ≥ 10. NICE and G-BA require TPS ≥ 50%. Different assays, different thresholds, different eligible populations. Without both pre-specified in Phase III, you cannot satisfy all three markets.

**03 — 100 days from JCA scope to dossier deadline**
Since January 2025, the EU HTAR (Health Technology Assessment Regulation) mandates JCAs (Joint Clinical Assessments) for all new oncology products. The scope document defining required evidence arrives only 100 days before the dossier deadline. Preparation must start years earlier, against precedents that haven't been published yet.

Today, answering these questions is a manual consulting engagement costing €50,000–200,000 over several weeks. Verum answers it with every inference traceable to a specific published HTA decision.

---

## How It Works — Four Steps

**01 — Mine decisions**
Every published HTA decision — Spanish IPTs, NICE STAs, G-BA dossiers, HAS decisions — is ingested and parsed. LLM extraction pulls the implicit PICO from each document, always linked to the source paragraph.

**02 — Structure evidence**
Extractions are validated by a human reviewer before being committed to the database. Every EvidenceUnit is versioned, auditable, and traceable — not a summarisation, a structured data object with a complete audit trail.

**03 — Map divergences**
Cross-country PICO matrix showing where agencies align and where they diverge — population, biomarker, comparator, endpoint, study design — including JCA consolidated scope as European reference layer.

**04 — Recommend action**
Strategic recommendations calibrated to your stage: Phase II→III protocol design, pre-launch market sequencing, dossier preparation, new indication planning, or competitive intelligence. The analysis tells you what to do next.

---

## Who It's For — Five Decision Moments

| Moment | Users | Value |
|---|---|---|
| Phase II → III | HEOR · Clinical Development | Lock the right protocol before the trial runs |
| Pre-launch | Market Access · HEOR | Sequence markets by evidence gaps, not assumptions |
| Dossier preparation | Regulatory Affairs · HEOR | Write what each agency expects, not a generic package |
| New indication | Clinical Development | Reuse existing evidence before commissioning new studies |
| Market intelligence | Market Access · Strategy | Track agency PICO signals as the corpus updates |

---

## The EvidenceUnit — Core Data Object

The minimum reusable evidence object. One EvidenceUnit can populate multiple country dossiers. Stored as JSONB in PostgreSQL/Supabase, indexed by indication composite key: `{drug_id, condition_code, subgroup_filter, line_of_therapy}`.

Schema aligned with GRADE evidence tables and PRISMA reporting standards.

| Field | Description | Example |
|---|---|---|
| `study_reference` | Source ID, database, study type | PubMed 38291045, RCT |
| `population_json` | PICO P: inclusion criteria, sample size | NSCLC stage III/IV, n=847 |
| `intervention_json` | PICO I: drug, dose, duration | Pembrolizumab 200mg Q3W |
| `comparator_json` | PICO C: SoC, active comparator | Platinum-based chemo |
| `outcome_json` | PICO O: endpoint, value, CI | OS HR 0.73 [0.58–0.92] |
| `evidence_quality` | GRADE level, risk of bias — human-assigned | High, low risk of bias |
| `hta_context_json` | Country, agency, applicable guidelines | ES, AEMPS, IPT criteria |
| `version` | Append-only audit trail — never overwrite | v1.2, 2026-01-15 |
| `extraction_meta` | LLM model, prompt version, confidence per field | claude-3, prompt-v3, OS: 0.91 |

---

## Human-in-the-Loop Design

Every pipeline phase produces a draft requiring human approval before the pipeline advances. The audit trail captures who approved, what version, when, and with what role.

This is not a constraint — it is the trust model. No AI tool in HTA has been accepted at submission level without full human accountability.

| Phase | Human task | Estimated effort |
|---|---|---|
| P1 — Ingestion | HEOR Scientist reviews retrieved study set; approves or refines query | ~1–2 hrs / indication |
| P2 — Structuring | Field-by-field EvidenceUnit review; GRADE assignment validated | ~3–5 min / unit |
| P3 — Localisation | Section mapping review; evidence gaps resolved; comparator confirmed | ~2–4 hrs / country |
| P4 — Output | Full dossier draft reviewed; citations verified; Medical Director sign-off | ~4–8 hrs / dossier |

---

## Regulatory Context

The EU HTAR entered into force January 2025. All new oncology products are subject to JCAs. The JCA implementing act requires SLR (Systematic Literature Review) searches no older than 3 months, with the final scope issued only 100 days before the submission deadline.

This compresses timelines brutally. Reusable, versioned EvidenceUnits are the architectural answer to a blank-page rebuild under time pressure.

---

## The Corpus

All sources are publicly available. The gap is not access — it is structure.

| Source | Volume | Notes |
|---|---|---|
| AEMPS IPTs (Spain) | 550+ total · 125/year · 60 oncology in 2024 | PDF parsing + OCR for older docs |
| NICE STAs (UK) | Part of 2,500+ EU decisions | Structured URLs, downloadable |
| G-BA dossiers (Germany) | Part of 2,500+ EU decisions | Public PDFs |
| HAS decisions (France) | Part of 2,500+ EU decisions | Public PDFs |
| PubMed / ClinicalTrials.gov | — | Public REST APIs, rate-limited |
| Embase | Licensed | Client-provided or CRO partnership |

---

## Tech Stack (Designed)

| Layer | Technology |
|---|---|
| Database | PostgreSQL / Supabase — JSONB, append-only EvidenceUnit records |
| Extraction | Claude / GPT-4 — structured PICO prompts, confidence scoring per field |
| Document output | python-docx, pandoc |
| Frontend | React + TypeScript + Tailwind CSS |
| Hosting | GitHub Pages (prototype) |

---

## What This Is Not

- Not an AI that writes HTA dossiers — output is structured evidence intelligence, not regulatory narrative
- Not a literature search engine — does not replace PubMed or Embase queries
- Not a substitute for HEOR expert judgement — informs decisions, does not make them
- Not a built product — validated concept with complete architecture and documented gap

---

## Related

**Evidence Mapper** — live AI-assisted pipeline over 63,000+ completed ClinicalTrials.gov trials. Built with the same HITL (Human-in-the-Loop) and structured data extraction principles that underpin Verum's architecture. [evidence-mapper.com](https://evidence-mapper.com)

---

María Castro · March 2026


---

## Related

[Evidence Mapper](https://evidence-mapper.com) — live AI-assisted pipeline over 63,000+ ClinicalTrials.gov completed trials. Built with the same HITL and structured data extraction principles that underpin Verum's architecture.

---

*María Castro · March 2026*
