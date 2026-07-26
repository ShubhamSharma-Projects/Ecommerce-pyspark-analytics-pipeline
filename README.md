# E-Commerce Transaction Analytics Pipeline (PySpark)

An end-to-end data pipeline built in PySpark that cleans, transforms, and segments over **1 million UK e-commerce transactions**, culminating in a window-function-based RFM (Recency, Frequency, Monetary) customer segmentation model and a partitioned Parquet output layer.

## Overview

This project processes the [Online Retail II](https://archive.ics.uci.edu/ml/datasets/Online+Retail+II) dataset , real transaction-level data from a UK-based online retailer spanning **2009–2011** through a full PySpark pipeline: ingestion, data quality cleaning, feature engineering, customer segmentation, and optimized storage.

Built and run on Databricks using PySpark's DataFrame API.

## Dataset

- **Source:** UCI Machine Learning Repository / Online Retail II
- **Raw size:** 1,067,371 transaction records
- **Fields:** Invoice, StockCode, Description, Quantity, InvoiceDate, Price, Customer ID, Country

## Pipeline Steps

### 1. Ingestion
Loaded the raw CSV into a PySpark DataFrame with schema inference and validated structure via `printSchema()`.

### 2. Data Cleaning
Applied a sequence of data quality filters to move from raw to analysis-ready data:

| Step | Rows Remaining |
|---|---|
| Raw data | 1,067,371 |
| After removing duplicate records | 1,033,036 |
| After removing null Customer IDs | 797,885 |
| After removing cancelled invoices & invalid quantity/price | **779,425** |

- Removed exact duplicate rows
- Dropped transactions with missing `Customer ID` (guest/unidentified checkouts)
- Excluded cancelled orders (invoices prefixed with `C`)
- Filtered out non-positive `Quantity` and `Price` values

### 3. Feature Engineering
Derived additional columns to support time-based and revenue analysis:

- `Revenue` = Quantity × Price
- `Year`, `Month`, `Month_Name`, `Quarter`, `Weekday`, `Hour` — extracted from `InvoiceDate`
- `Weekend` flag (Weekday vs. Weekend)
- `InvoiceMonth` — truncated to month for time-series grouping

### 4. Customer Segmentation (RFM, via Window Functions)
Built a customer-level RFM model using PySpark's `Window` and `ntile()` functions:

- **Recency** — days since each customer's last purchase (relative to the latest invoice date in the dataset)
- **Frequency** — count of distinct invoices per customer
- **Monetary** — total revenue per customer

Each customer was scored 1–5 on each dimension via `ntile(5)` over ranked windows, combined into an `RFM_Score`, and mapped into business-relevant segments:

| Segment | Customers |
|---|---|
| At Risk | 2,350 |
| Potential Loyalists | 1,591 |
| Loyal Customers | 1,222 |
| Others | 479 |
| Champions | 236 |

### 5. Optimized Storage
Wrote the final transformed dataset to **partitioned Parquet** (partitioned by `Year` and `Month`), enabling efficient downstream filtering  validated by querying a single month's partition directly (e.g., January 2011: 20,988 rows) without scanning the full dataset.

## Tech Stack

- **PySpark** (DataFrame API, Window functions, `ntile`)
- **Databricks** (execution environment)
- **Parquet** (partitioned columnar storage)

## Key Results

- Cleaned and validated **779K+ transaction records** from a raw 1M+ row dataset, removing duplicates, guest checkouts, cancellations, and invalid entries
- Segmented **5,878 unique customers** into 5 actionable tiers using window-function-based RFM scoring
- Identified **236 "Champion" customers** and **2,350 "At Risk" customers**, directly actionable for retention vs. reactivation strategy
- Reduced downstream query scope via Year/Month-partitioned Parquet output

## How to Run

1. Download the [Online Retail II dataset](https://archive.ics.uci.edu/ml/datasets/Online+Retail+II) and place it as a CSV.
2. Update the file path in the ingestion cell to point to your dataset location.
3. Run the notebook top to bottom in a PySpark-enabled environment (Databricks Community Edition or a local Spark session).

