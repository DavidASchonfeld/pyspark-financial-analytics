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

**Step 7 — Window Functions**
Answers questions like "what is the most recent filing for each company?" using a technique called window functions. Unlike GROUP BY functions (which discard details to produce totals), window functions let each record keep all of its original information while also gaining a rank, a running count, or any other comparison relative to its group.

**Step 8 — GroupBy Aggregation**
Ranks companies by how frequently they file with the SEC, producing a leaderboard-style summary with metrics like total filings, how many distinct form types they submitted, and how far back their filing history goes.

**Step 9 — Pure SQL in Spark**
Re-expresses the Step 8 aggregation using standard SQL instead of the PySpark DataFrame API — producing identical results. Spark lets you register any DataFrame as a temporary SQL view and query it just like a database table, which is useful for collaborating with SQL-first teammates, situations, etc.

**Step 10 — Saving to Disk (Partitioned Parquet)**
Writes the final dataset to disk in Parquet format — the industry-standard file format used across data engineering. Unlike a CSV, Parquet stores data column-by-column, compresses repeated values automatically, and supports *partitioning*: physically splitting the output into separate folders (one per filing year) so that future queries touching only one year never read the others. I also demonstrate *partition pruning* — the automatic performance boost where Spark skips irrelevant folders entirely before reading a single file — and verify it is working by inspecting Spark's internal execution plan with `.explain()`.


## Tech Stack

- **PySpark** — distributed data processing
- **SEC EDGAR API** — live U.S. public company filing data
- **Python 3** — orchestration and business logic
- **Jupyter Notebook** — interactive, step-by-step execution
