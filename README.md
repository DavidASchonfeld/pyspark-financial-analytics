# PySpark Financial Analytics

A hands-on data engineering project that pulls real financial filing data from the SEC EDGAR API (the U.S. government's official public database of company filings) and processes it using PySpark — the industry-standard tool for working with large datasets.

## What It Does

The notebook walks through a full data pipeline, step by step:

**Step 1 — Configuration**
Sets up the Spark environment with sensible defaults so that data previews and logging work consistently throughout the rest of the notebook.

**Step 2 — Ingest Data**
Fetches live filing records and company information from the SEC EDGAR API, loading them into Spark DataFrames ready for processing. All columns are kept as plain text at this stage to avoid losing rows during ingestion.

**Step 3 — Type-Cast**
Converts raw text columns into proper data types (dates, numbers, categories) so that downstream calculations and filters work correctly.

**Step 4 — Deduplicate**
Removes null values and duplicate rows, ensuring the dataset is clean and trustworthy before any analysis begins.

**Step 5 — Enrich**
Adds derived fields — such as filing quarter, days since filing, and a human-readable report type label — that make the data more useful for analysis without touching the original source values.

**Step 6 — Join Companies & Filings**
Links each filing record to its corresponding company metadata (name, ticker, industry) using an efficient broadcast join, producing a single unified table ready for querying.

## Tech Stack

- **PySpark** — distributed data processing
- **SEC EDGAR API** — live U.S. public company filing data
- **Python 3** — orchestration and business logic
- **Jupyter Notebook** — interactive, step-by-step execution
