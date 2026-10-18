# GrantScopeAI

### From a research idea to evidence from previously funded projects

**GrantScopeAI** is a research-intelligence and decision-support project for exploring AI-enabled chemistry and materials research. It combines historical grant records from the **U.S. National Science Foundation (NSF)** and **CORDIS** with publication-activity context from **OpenAlex**.

A researcher can describe an early-stage idea in plain language, find comparable funded projects, inspect *why* they matched, shortlist and compare examples, and explore relevant research areas, recipient organisations, and funding programmes. A companion **Tableau dashboard** shows the wider funding and publication landscape.

**Explore the project**

- **Live Streamlit application:** *[Add your Streamlit Community Cloud URL]*
- **Interactive Tableau dashboard:** *[Add your Tableau Public URL]*
- **Source code and notebooks:** [GitHub repository](https://github.com/Orgo4ever/GrantScope)

**Tech stack:** Python · pandas · NumPy · scikit-learn · TF-IDF · cosine similarity · Streamlit · Tableau · Git/GitHub

> **Scope:** This is an exploratory tool built from **2021–2025 historical data**, not a live grant-opportunity service or a model that predicts funding success.

## Project at a glance

| Measure | Result |
|---|---:|
| Integrated NSF and CORDIS funding records | **3,339** |
| NSF awards | 2,664 |
| CORDIS projects | 675 |
| Documents in the similarity-search catalogue | **2,943** |
| Research topics in the shared taxonomy | 8, plus a fallback category |
| OpenAlex topic-year observations | 40 |
| Data-quality checks | **19 passed; 1 informational review** |
| Analysis period | 2021–2025 |

*The 2,943 searchable documents are the model's deduplicated retrieval catalogue; the integrated funding dataset retains 3,339 source records.*

## The problem

Researchers and research-support teams often need to consult separate grant databases, funding websites, and publication indexes to understand the landscape around a new idea. That makes it difficult to answer connected questions:

- Has comparable work already received funding?
- Which organisations and funding programmes appear in related projects?
- How has funding activity changed across research areas and years?
- Is publication activity growing alongside grant activity?
- Which historical projects are useful starting points for further investigation?

**GrantScopeAI brings these signals into a common analytical workflow.** Its purpose is to support research orientation and evidence-based exploration—not to assess proposal eligibility, guarantee novelty, or estimate the probability of obtaining a grant.

## What I built

### 1. An interactive research-discovery app (Streamlit)

The application supports a four-step workflow:

1. **Describe a research concept.** Enter a natural-language description and optionally filter by funding source, topic, award year, or number of results.
2. **Review similar funded projects.** Inspect ranked NSF/CORDIS records with a relevance score, matching explanation, shared concepts, project summary, funding metadata, and link to the original source.
3. **Save and compare.** Add projects to a session shortlist, compare up to three side-by-side, and download the shortlist or comparison as CSV.
4. **Explore starting points.** See related topics, recipient organisations, funding programmes, and historically funded projects based on the current results.

Relevance is a **model-specific similarity score**, not a funding-success probability. The app also explains the limitations of historical grant data.

### 2. A funding-landscape dashboard (Tableau)

The companion dashboard lets users explore:

- Unique funded-project counts and represented research topics.
- NSF funding (**USD**) and CORDIS funding (**EUR**), shown separately.
- Annual grant activity by topic.
- Grant counts by research topic and funding source.
- Topic momentum: grant activity in the context of publication growth.
- Leading funding programmes and recipient organisations.

Interactive selections filter compatible views by topic, year, or organisation. The dashboard uses several prepared datasets at different levels of aggregation, so some fixed-period measures intentionally do **not** respond to every filter.

### 3. An end-to-end analytical pipeline

The project covers public-data acquisition, cleaning, harmonisation, feature engineering, validation, exploratory analysis, similarity ranking, and delivery through interactive tools.

```text
NSF Awards API ---------┐
                        ├─> Clean + harmonise grants ─> Integrated grant table
CORDIS project files ---┘                                  │
                                                           ├─> Tableau analytical outputs
OpenAlex API ─> Topic/year publication activity ──────────┘
                                                           │
Integrated grant text ─> TF-IDF + hybrid ranking ─> Search catalogue
                                                           │
Research concept ──────────────────────────────────> Streamlit app
```

NSF and CORDIS are **appended**, not joined, because they describe different funded projects and do not share a common project identifier. OpenAlex remains at the **topic-year** level rather than being treated as another source of individual grants.

## Data sources and preparation

| Source | Role | Processing |
|---|---|---|
| **NSF Awards API** | U.S. award records | Monthly queries and pagination; duplicate award records consolidated; fields cleaned and standardised. |
| **CORDIS** | EU-funded project and organisation data | Project, participant/organisation, and research-vocabulary files combined; participant-level information aggregated to the project level. |
| **OpenAlex API** | Publication activity by research topic | Topic-year counts and growth indicators prepared separately from grant-level records. |

The raw NSF extraction reached **33,125 rows across 70 columns** before relevance filtering and subsequent processing. The final harmonised funding table contains **3,339 unique records across 25 application-ready fields**.

### Relevance filtering

Funding records were assessed using AI and chemistry/materials keyword rules independent of the original extraction-query labels:

| Source | `core_match` | `broad_match` | Total candidates |
|---|---:|---:|---:|
| CORDIS | 416 | 259 | 675 |
| NSF | 1,757 | 907 | 2,664 |
| **Combined** | **2,173** | **1,166** | **3,339** |

- **`core_match`**: Records satisfying the more selective relevance criteria.
- **`broad_match`**: Potentially useful records retained by broader, higher-recall criteria.
- **`out_of_scope`**: Records kept in the full local extracts but excluded from the candidate dataset.

Source identifiers and original URLs were retained for traceability. Unique integrated keys use prefixes such as `NSF_<award_id>` and `CORDIS_<project_id>`.

### Shared research-topic taxonomy

A transparent, priority-based taxonomy groups the funding records into eight AI-enabled chemistry/materials areas:

1. Autonomous laboratories
2. Reaction prediction
3. AI-enabled catalysis
4. Materials informatics
5. Cheminformatics
6. Molecular machine learning
7. AI-enabled materials
8. AI-enabled chemistry

Rules use available grant titles, descriptions, programmes, and related metadata. Each grant receives a `primary_topic` as well as any additional matched topics.

- **2,239** grants matched at least one named topic.
- **3,245** grant-topic relationships were identified.
- **1,100** grants remained in **Other AI-enabled chemistry/materials** rather than receiving an unsupported specific label.

Because one grant can match multiple topics, topic-level totals should not be summed and interpreted as unique grants.

### Topic-year and publication context

The funding pipeline builds an **80-row source × topic × year grid** (2 funding sources × 8 topics × 5 years), retaining zero-activity combinations. A separate **40-row topic-year context table** aligns grant activity with OpenAlex publication counts and growth metrics.

The OpenAlex table contains eight topics across 2021–2025, with no duplicate topic-year combinations or missing/negative publication counts. Indicators include year-over-year growth, a 2021 baseline index, 2021–2025 total growth, and compound annual growth rate.

OpenAlex topic searches can overlap, so publication counts are compared **within topics** rather than summed into a single cross-topic publication total. Publication growth indicates research activity; it does not establish research quality or future funding availability.

## Data quality and governance

Data quality was treated as a core deliverable, not an afterthought. The integrated validation framework contains **20 rules** addressing:

- Row-count reconciliation and globally unique grant identifiers.
- Completeness and consistency of analytical fields.
- Dates, funding values, currencies, and source URLs.
- Relevance-tier and taxonomy values.
- Duplicate grant-topic relationships.
- Complete OpenAlex topic-year coverage and cross-source taxonomy alignment.

**Validation result: 19 passed; one informational item flagged for review.** The review concerns grants deliberately retained in the fallback topic category, not a failed data-processing operation.

A data dictionary and source-to-target schema crosswalk document field definitions, data types, availability, transformation logic, nullability, and interpretation constraints.

Two design decisions are especially important:

- **Preserve funding definitions.** NSF uses estimated total awards in **USD**; CORDIS uses EU contributions in **EUR**. The dashboard does not sum these into a misleading combined amount.
- **Preserve analytical grain.** Grant-level data, organisation/programme summaries, and topic-year publication measures remain separate where combining them would create duplication or imply unsupported comparisons.

## Similarity model: explainable retrieval, not prediction

The final search system uses **normalised, title-weighted TF-IDF**, **cosine similarity**, and additional context-aware ranking signals. These signals consider scientific domain, chemistry/reaction context, research workflow, and clearly unrelated application areas. Similar project-title families are diversified so closely related award records do not overwhelm the results.

A user's concept is transformed into the fitted TF-IDF feature space, compared with historical funded projects, and ranked. Result cards present the score alongside readable evidence for the match.

### Model iteration and evaluation

Five approaches were explored:

1. Initial TF-IDF.
2. Normalised, title-weighted TF-IDF.
3. Concept-aware hybrid ranking.
4. Field-aware hybrid ranking.
5. **Context-aware hybrid ranking (selected).**

On a small, manually reviewed **eight-project demonstration benchmark** (seven relevant examples and one weak comparison), the selected approach showed more useful ranking separation:

- **3 of 7** relevant examples ranked in the top 25.
- **5 of 7** ranked in the top 50.
- The best strong match moved from **rank 54 to rank 14**.
- The weak drug-discovery comparison moved down to **rank 212**.
- All seven relevant examples ranked above that weak comparison.

Additional queries covering catalyst discovery, sustainable polymers, and autonomous reaction laboratories produced distinct, explainable project results. The packaged catalogue, vectoriser, sparse matrix, configuration, and evaluation outputs also passed **eight reload/validation checks**.

**Evaluation caveat:** This was a small, query-specific relevance check, **not** a statistically representative accuracy or production-performance estimate. The tool has no reliable dataset of rejected proposals and therefore does **not** attempt to predict funding success.

## Exploratory findings

The 2021–2025 analysis suggested several patterns **within the selected dataset and research taxonomy**:

- **AI-enabled materials and chemistry** had the largest combined funding and publication footprints.
- **AI-enabled catalysis** combined substantial grant activity with strong publication momentum.
- **Autonomous laboratories** showed particularly rapid *relative* publication growth while remaining smaller in absolute funding activity.
- **Materials informatics and reaction prediction** showed publication growth alongside comparatively lower recent grant activity.
- NSF and CORDIS differed in programme structure, organisation coverage, geography, and award-value definitions.

These are descriptive, dataset-specific insights. They are not causal findings or forecasts of future awards.

## Repository guide

```text
GrantScope/
├── App_Advanced_Features.py    # Main Streamlit application
├── App.py                      # Earlier application version
├── requirements.txt            # Streamlit Cloud / local dependencies
├── src/                        # Model-loading and search logic
├── Notebooks/                  # Acquisition, cleaning, analysis, modelling
├── Data/                       # Compact processed analytical datasets
├── Models/                     # Saved retrieval model assets
├── Tableau Files/              # Tableau workbook and related files
├── Visualizations/             # Exported visual assets
└── Presentations/              # Project presentation materials
```

Key notebooks include `Notebooks/02_cordis_cleaning.ipynb`, `Notebooks/03_nsf_cleaning.ipynb`, `Notebooks/04_openalex_preparation.ipynb`, and `Notebooks/07_similarity_model.ipynb`.

Large raw/enriched extracts are **not** included in the public repository. Compact processed datasets and the deployed app's required model assets are provided where appropriate; original records can be consulted using retained source links.

## Run the app locally

Clone the repository, install the dependencies, and start the Streamlit application:

```bash
git clone https://github.com/Orgo4ever/GrantScope.git
cd GrantScope
python -m pip install -r requirements.txt
python -m streamlit run App_Advanced_Features.py
```

Use a virtual environment if preferred. The app expects the saved model assets and accompanying files to remain in their repository-relative locations. The public Streamlit deployment uses the same application entry point.

## Limitations and responsible use

- **Historical scope:** Records cover 2021–2025; historical programmes are not necessarily open now.
- **Not a funding prediction:** Ranking scores indicate model-specific similarity, not the probability of receiving funding.
- **Different funding conventions:** NSF estimated awards and CORDIS EU contributions are not directly interchangeable, and their currencies are not aggregated.
- **Metadata quality:** Public descriptions, organisation names, and programme information may be incomplete or inconsistent.
- **Topic simplification:** Rule-based categories support exploration but cannot fully represent interdisciplinary research.
- **Lexical retrieval:** TF-IDF is efficient and interpretable but may miss semantically related projects expressed with different terminology.
- **Comparability:** Publication topics can overlap; dashboard views use different grains and have intentionally limited cross-filtering.
- **Independent verification:** Users must check current programme eligibility, deadlines, award conditions, and scientific novelty against original sources.

Possible future work includes larger-scale retrieval evaluation, semantic embedding comparisons, and a separately verified layer for currently open funding opportunities.

## Project context

**Project type:** Independent final project for the Ironhack Data Analytics Bootcamp (2026).  
**Focus:** Data integration, quality validation, exploratory analytics, explainable information retrieval, and decision-support visualisation.  
**Author:** Kyle Haustein.

The project demonstrates the complete path from fragmented public datasets to a validated analytical model and an interactive tool designed around a real research-support workflow.
