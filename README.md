# California Residential — MLS Analytics Pipeline

A reproducible data-analysis pipeline over **California residential** MLS
data (Sold & Listing extracts, ~28 months, Jan 2024 – Apr 2026). It ingests
monthly data from the CoreLogic **Trestle** API, cleans and feature-engineers it
through an ordered set of stages, flags outliers with IQR, and prepares lean
tables for **Tableau** dashboards.

> Built during a data-analyst internship at IDX Exchange.

**Confidentiality:** the underlying MLS data is confidential and is **not**
included in this repo — the CSV extracts, `.twbx` workbooks, and the API
extraction scripts are gitignored.

---

## Pipeline

Stages run in dependency order (Sold and Listed chains stay in sync):

| Stage | Script | What it does |
|---|---|---|
| 1 | `week1_*.py` | Concatenate monthly CSVs, filter to `PropertyType == 'Residential'` |
| 2–3 | `week2_3_*.py` | EDA + validation + FRED 30-year fixed mortgage-rate enrichment |
| 4–5 | `week4_5_*.py` | Date/numeric cleaning, business-rule flags, geo checks |
| 6 | `week6_*.py` | Feature engineering (price ratios, $/sqft, timeline metrics) |
| 7 | `week7_*.py` | IQR outlier flagging → `*_flagged` + `*_clean` datasets |
| 8 | `week8_tableau_prep.py` | Lean row-level extracts + pre-aggregated monthly tables for Tableau |
| — | `eda_report.py` | Self-contained HTML EDA report (charts + findings) |

Run the whole thing with the orchestrator:

```bash
python run_pipeline.py            # full pipeline
python run_pipeline.py --from week6   # resume from a stage
python run_pipeline.py --list     # list stages
```

## Data ingestion

- **Trestle (CoreLogic)** — monthly Sold & Listing property extracts via the
  OData REST API (`mls_sold.py` / `mls_listed.py`, gitignored).
- **FRED (St. Louis Fed)** — 30-year fixed mortgage rate, merged onto each
  month for the rate/price analysis (`connectors/fred_connector.py`).

## Dashboards

Tableau workbooks (`market_analysis.twbx`, `competitive_analysis.twbx`,
gitignored) are built on the Week 8 extracts. See
[`TABLEAU_GUIDE.md`](TABLEAU_GUIDE.md) for the dashboard build notes.

## Setup

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
cp .env.example .env          # set FRED_API_KEY
```

## Stack

`Python` · `pandas` / `numpy` · `matplotlib` · `Tableau` · REST APIs
(CoreLogic Trestle, FRED)
