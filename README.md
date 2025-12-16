## Project Overview

**Medicare Spending per Beneficiary (MSPB)** is a CMS metric that compares how much Medicare spends on a hospital stay relative to the national average.

* MSPB = **1.00** → Equal to national average  
* MSPB < **1.00** → More cost-efficient  
* MSPB > **1.00** → Higher spending than average  

This project combines **CMS MSPB data** with **Hospital General Information** to evaluate whether higher spending correlates with better hospital quality, ownership type, or geographic location.

---

## Medicare Spending per Beneficiary (MSPB)

This repository contains an end-to-end **Healthcare Analytics / Business Intelligence** project that analyzes **Medicare Spending per Beneficiary (MSPB)** across U.S. hospitals. The goal is to evaluate hospital cost efficiency, identify key drivers of spending variation, and deliver an executive-ready Power BI dashboard.

This project demonstrates a full analytics workflow: **data ingestion → cleaning/validation → SQL Server modeling → Power BI semantic layer (DAX) → executive dashboard + insights**.

---

## What is MSPB?

**Medicare Spending per Beneficiary (MSPB)** is a CMS metric that compares the Medicare spending for a hospital episode relative to the national average.

* MSPB = **1.00** → Equal to national average spending  
* MSPB < **1.00** → More cost-efficient (spending below average)  
* MSPB > **1.00** → Higher spending than average  

This project combines **CMS MSPB** with **Hospital General Information** to assess whether spending varies by **state**, **hospital ownership**, and **hospital quality ratings**.

---

## Business Questions Answered

* How many hospitals are in the dataset and what is the overall MSPB average?
* How many hospitals spend **above** vs **below** the national average (MSPB > 1 vs < 1)?
* Which states have the highest and lowest average MSPB?
* Does higher Medicare spending correlate with better hospital quality ratings?
* How does MSPB vary by hospital ownership type?
* Which hospitals are the most extreme outliers (highest spending vs most cost-efficient)?

---

## Data Sources

* CMS Medicare Spending per Beneficiary (MSPB)
* CMS Hospital General Information
* CMS Provider Data Catalog

All datasets are publicly available from CMS Provider Data.

---

## Tech Stack (Detailed)

This project intentionally uses tools that mirror real-world analytics/engineering workflows.

### Python (Data Ingestion + ETL)

* **pandas**  
  * Load and clean raw CSV files  
  * Handle missing values, data types, and column normalization  
* **numpy**  
  * Numeric operations and feature calculations  
* **urllib**  
  * Download/transfer CMS datasets directly (or handle remote CSV retrieval)  
* **pydoc**  
  * Quick inspection and help documentation during development/debugging  
  * Useful for verifying function behavior and library usage while building ETL scripts  
* **SQLAlchemy**  
  * Database connection layer for SQL Server  
  * Used to push cleaned DataFrames into SQL Server tables reliably  
* **ODBC Driver / Connection String**  
  * Secure connection to SQL Server (local or remote instance)  
  * Supports repeatable ingestion pipelines  

### SQL Server (Data Modeling + Transformations)

* **Staging tables** for raw ingested data
* **Views** for reusable transformation logic
* **CTEs (Common Table Expressions)** for clean multi-step transformations
* **Window functions** (`ROW_NUMBER() OVER (PARTITION BY ...)`) for deduplication and quality-based record selection
* **TRY_CONVERT / CAST** for robust type conversion
* **Data quality checks** (null checks, duplicate checks, validation counts)
* **KPI summary views** (pre-aggregated metrics for Power BI)

### Power BI (Semantic Layer + Dashboard)

* **DAX Measures**
  * KPI metrics (Total Hospitals, Avg MSPB, Above/Below National Average counts)
  * Proper formatting (avoid K abbreviations, correct data types)
  * Context handling (ALL / ALLSELECTED when needed)
* **Visual Analytics**
  * Executive KPI cards
  * Filled map for state-level MSPB averages
  * Bar charts for ownership and quality rating comparisons
  * Ranked tables for top 10 highest-spending and most cost-efficient hospitals
* **Interactive filtering**
  * Slicers and cross-filtering between visuals

---

## End-to-End Pipeline (How the Data Flows)

### Step 1: Acquire CMS Data

* Download MSPB and Hospital General Info datasets from CMS
* Store raw files in a structured folder layout (raw vs processed)

### Step 2: Clean + Validate in Python

* Standardize column names and formats
* Identify missing values and non-numeric values (example: ratings like `Not Available`)
* Convert fields to correct types (numeric MSPB, integer ratings, categorical ownership)
* Validate row counts before/after cleaning to ensure no accidental data loss
* Export cleaned datasets for loading into SQL Server

### Step 3: Load into SQL Server (Automated)

* Use **SQLAlchemy** connection engine to load cleaned CSV outputs into SQL Server tables
* Benefits:
  * Repeatable ingestion (re-run pipeline without manual steps)
  * Faster refresh than manual import
  * Consistent schema enforcement

### Step 4: Model and Transform in SQL Server

* Build views for:
  * MSPB features (cleaned metrics + derived fields)
  * Hospital information deduplication (remove duplicate facility records)
  * Joined/enriched views combining MSPB + Hospital attributes
* Use window functions to control duplicates:
  * Keep the “best” row per facility based on data completeness

### Step 5: Power BI Dashboard + Measures

* Connect Power BI to SQL Server views (clean model layer)
* Create measures for:
  * Total hospitals
  * Average MSPB
  * Above / below national average counts
* Build executive visuals and outlier tables for insight

---

## Data Modeling Highlights (Recruiter-Friendly)

### Deduplication Strategy (Why it mattered)

CMS hospital tables may contain duplicates per Facility ID. If not handled, joins can inflate row counts and distort KPIs.

This project uses a **quality-based deduplication rule**:

* Partition by Facility ID
* Rank records by completeness (ownership present, rating present)
* Keep only `rn = 1`

This prevents:
* Duplicate joins
* Inflated counts and averages
* Misleading dashboard results

### KPI Engineering

The report includes a dedicated KPI dataset (example: `v_mspb_kpis`) to ensure:
* One-row KPI table for fast reporting
* Measures return exact values (e.g., **2899**, not **2.899K**)
* Reliable executive summary metrics

---

## Dashboard Pages (What’s Included)
<img width="1296" height="732" alt="image" src="https://github.com/user-attachments/assets/8de62831-c934-4ca6-82fb-7c132afb756a" />
<img width="1296" height="724" alt="image" src="https://github.com/user-attachments/assets/396ea5f8-8d24-43cf-847b-56febea054aa" />



### Executive Overview

* Total Hospitals (e.g., 2899)
* Average MSPB (e.g., 0.99)
* Count Above National Average
* Count Below National Average
* Filled map showing average MSPB by state

### Cost Drivers & Spending Outcomes

* Avg MSPB by Hospital Quality Rating
* Avg MSPB by Hospital Ownership
* Top 10 Highest-Spending Hospitals
* Top 10 Most Cost-Efficient Hospitals
* Key takeaway summary text for executive audiences

---

## Key Insights (Example Takeaways)

* MSPB varies significantly by state, indicating geographic cost variation
* Higher spending does not always correspond to better quality ratings
* Ownership type shows meaningful differences in spending patterns
* A small set of hospitals drive extreme over- and under-spending outliers

---

## Repository Structure (Suggested)

* `/python` – Data ingestion + cleaning scripts (urllib, pandas, SQLAlchemy)
* `/sql` – SQL Server views, dedup logic, KPI views
* `/powerbi` – Power BI report files and screenshots
* `/data` – Raw and cleaned datasets (if allowed to share)
* `README.md` – Documentation

---

## How to Run (High-Level)

* Run Python ETL scripts to download/clean data and load to SQL Server
* Execute SQL scripts/views to build the reporting layer
* Open Power BI file and refresh from SQL Server

---

## Future Enhancements

* Add year-over-year MSPB trends (if multi-year data is included)
* Add additional CMS quality measures to test spending vs outcomes deeper
* Build predictive model features for identifying cost drivers
* Add drill-through pages for hospital-level detail analysis

---

## Author

**Kishor Khatiwada**  
Data Analytics | Business Intelligence | Healthcare Analytics

