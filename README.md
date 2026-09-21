# Cloud Data Engineering Pipeline for Retail Sales Analytics

An end-to-end cloud ETL (Extract, Transform, Load) pipeline that cleans, transforms, and loads retail sales data into Azure SQL Database, built to explore the core data engineering workflow used across most cloud platforms: **raw data → cleaning → storage → transformation → structured database → dashboard.**

## Overview

This project takes a raw retail sales dataset, cleans it in Python, and moves it through a real Azure cloud pipeline into a database that's ready for analysis in Power BI. It was built as a hands-on way to learn the Azure data stack (Data Lake Storage, Data Factory, SQL Database) commonly used in cloud data engineering roles.

## Architecture

```
CSV (Kaggle dataset)
      ↓
Python (cleaning & transformation)
      ↓
Azure Data Lake Storage Gen2 (raw-data / cleaned-data containers)
      ↓
Azure Data Factory (Copy Activity pipeline)
      ↓
Azure SQL Database (sales_fact table)
      ↓
Power BI (dashboard)
```

## Tools & Technologies

- **Python** (pandas) — data cleaning and transformation
- **Azure Data Lake Storage Gen2** — landing zone for raw and cleaned data
- **Azure Data Factory** — orchestrates the pipeline via a Copy Activity
- **Azure SQL Database** — structured storage for the cleaned dataset
- **Power BI** — dashboard and visual analysis

## Dataset

This project uses the [Superstore Sales Dataset](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final) from Kaggle (~9,994 rows of retail order data: sales, profit, region, category, and customer details). The raw CSV is not included in this repo — download it directly from the Kaggle link above if you want to reproduce this pipeline.

## What the pipeline does

1. **Extract & Clean (Python)** — loads the raw CSV, fixes date types, standardizes column names to snake_case, checks for nulls/duplicates, and adds a `profit_margin` derived column.
2. **Load to Storage** — uploads the raw file to a `raw-data` container and the cleaned file to a `cleaned-data` container in Azure Data Lake Storage Gen2.
3. **Orchestrate (Data Factory)** — a Copy Activity pipeline reads the cleaned CSV and loads it into the `sales_fact` table in Azure SQL Database, with an explicit column mapping between CSV headers and SQL column names.
4. **Analyze (Power BI)** — connects directly to Azure SQL Database to build sales, profit, and regional performance visuals.

## Notable challenges solved

Building this surfaced a few real data engineering issues worth mentioning:

- **Encoding mismatches** — the raw CSV required `ISO-8859-1` encoding to load correctly in pandas.
- **CSV quoting/escaping** — several product names contain embedded commas and quote marks (e.g. `14 7/8" x 11"`), which broke Data Factory's default parser until the file was re-exported with full quoting (`csv.QUOTE_ALL`) and the dataset's escape character was explicitly set to match.
- **Schema mapping** — CSV headers (`Order ID`) didn't match SQL's snake_case columns (`order_id`), requiring manual column mapping in the Data Factory Copy Activity rather than relying on auto-mapping.

## Setup / reproduction

1. Create an Azure account (Azure for Students works well — free credit, no card required).
2. Create a Resource Group, a Storage Account (with hierarchical namespace enabled for Data Lake Gen2), an Azure SQL Database, and a Data Factory instance.
3. Run the cleaning notebook in `notebooks/` on the raw Kaggle CSV.
4. Upload the cleaned CSV to your storage account's `cleaned-data` container.
5. Run the SQL script in `sql/create_table.sql` in your database's Query editor to create the `sales_fact` table.
6. Build a Data Factory pipeline with a Copy Activity: source = your CSV dataset, sink = `sales_fact`.
7. Connect Power BI Desktop to your Azure SQL Database and load `sales_fact`.

## Status

**In progress** — Python cleaning, Azure Storage, Data Factory, and Azure SQL Database are complete. Power BI dashboard is currently in development.
