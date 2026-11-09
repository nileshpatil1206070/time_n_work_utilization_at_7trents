

# 7-Trent Operational Workforce Analytics Pipeline

An end-to-end Lakehouse and reporting solution built with **Databricks**, **Delta Lake**, and **Power BI** to track frontline workforce utilisation, sickness, and job planning adherence.

---

### 1. Problem Statement
Utility operations require visibility into frontline efficiency and schedule tracking:
* **Utilisation Rate:** Are contracted shift hours being effectively used for work?
* **Sickness %:** What percentage of total available shift hours is lost to sickness?
* **Plan Adherence:** Were work orders planned on Day 1 actually completed as new daily deltas arrived?

---

### 2. Architecture & Pipeline Workflow

Data Lakehouse Architecture Pipeline (Medallion Pattern)

1. Source Layer

-Databricks Unity Catalog Volumes (raw_data/) storing daily batch CSV folders (DD_MM_YYYY/).

-Contains initial full loads and daily delta change files.

2. Bronze Layer (Raw Ingestion)

-Dynamic folder discovery and idempotent append ingestion.

-Adds audit lineage: _source_file, _batch_date, _ingested_at, and _load_type (FULL_LOAD vs DELTA).

-Raw Delta tables: fact_work, fact_shift, fact_shrinkage, and 4 dimension tables.

3. Silver Layer (Cleansing & Snapshots)

-Data type casting, null handling, and duration calculations (shift_hours, planned_hours, shrinkage_hours).

-Deduplication of shift and absence events.

-Reconstructs silver.fact_work_snapshot to capture daily point-in-time job statuses for plan tracking.

4. Gold Layer (Kimball Star Schema)

-Conformed Dimensions: dim_date, dim_people, dim_county, dim_cancellation_reason, and dim_shrinkage_type.

Analytical Facts:

-fact_shift: Granular shift tickets and hours.

-fact_shrinkage: Granular absence events.

-fact_work_snapshot: Cumulative daily job status snapshots.

-fact_shift_shrinkage_daily: Unified operative-day capacity table to prevent join fan-out.

5. Consumption Layer (Power BI)

-Direct connection via Databricks SQL Warehouse.

-Dynamic DAX measures:

-Utilisation Rate %: (Planned Hours + Shrinkage Hours) / Shift Hours

-Sickness %: Sickness Hours / Shift Hours

-Plan Adherence %: Completed jobs on evaluation date vs planned jobs on baseline date.

-Features: Executive KPI cards, regional breakdown, and operative drill-through pages.


6. Project Team Roles
Ankur: Raw data landing and folder setup in Unity Catalog Volumes,Bronze layer dynamic batch ingestion and audit columns.
Raju: Silver layer transformations and Dated Periodic Snapshot logic.
Nilesh: Gold layer dimensional modeling
Nivedita: Power BI dashboard, and DAX KPIs.

8. Execution Steps
Run 01_bronze_ingestion to dynamically load raw daily batch folders into Bronze Delta tables.
Run 02_silver_transformations to clean records and build silver.fact_work_snapshot.
Run 03_gold_dimensions and 04_gold_facts to materialize Gold star schema tables.Open the Power BI Desktop file, connect to Databricks Gold tables, and refresh the report.
