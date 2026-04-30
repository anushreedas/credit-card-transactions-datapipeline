# Credit Card Transactions Data Warehouse

An ETL pipeline and dimensional data warehouse for credit card transaction analysis, built in R. This project transforms raw credit card data into a star schema optimized for fraud detection, customer behavior analysis, and business intelligence reporting.

## Project Overview

This project was completed in two stages:

- **Part 1** — Star schema design and initial ETL pipeline
- **Part 2** — Advanced dimensional modeling and data quality assessment

## Dataset

Source: [Credit Card Transactions — IBM (Kaggle)](https://www.kaggle.com/datasets/ealtman2019/credit-card-transactions/data)

Three raw CSV files are used as input:

| File | Description |
|---|---|
| `credit_card_transactions-ibm_v2.csv` | Transaction records (24M+ rows) |
| `sd254_cards.csv` | Credit card details |
| `sd254_users.csv` | Cardholder information |

## Schema Design

![Star Schema](https://github.com/anushreedas/credit-card-transactions-datapipeline/blob/main/images/updated_star_schema.png)

### Fact Table
- **Transactions** — `TransactionId`, `UserId`, `CardId`, `DateId`, `MerchantId`, `AmountCategory`

### Dimension Tables
- **Users** — Demographics, income, debt, FICO score
- **Cards** — Card brand, type, credit limit, expiration
- **Merchants** — Name, location, merchant category code (MCC)
- **Date** — Year, month, day, time of day category
- **Errors** — Error type descriptions (normalized from transaction data)

### Junk Dimension
- **Transaction Flags** — `UseChip`, `IsFraud`

### Bridge Table
- **Transaction–Error Bridge** — Resolves the many-to-many relationship between transactions and error types

## ETL Pipeline

### Extract
Load raw CSV files into R dataframes.

### Transform

**Data Cleaning & Structuring**
- Add surrogate primary keys where missing
- Rename columns to a consistent format
- Remove duplicate rows
- Convert columns to correct data types (factors, dates, numeric, character)
- Strip currency symbols and parse numeric fields

**Dimensional Modeling**
- Extract Merchants and Date dimension tables from the transactions table to eliminate redundant data
- Create a Junk Dimension for transaction flag columns (`UseChip`, `IsFraud`)
- Normalize the `Errors` column (a delimited list) into a dedicated Errors dimension and bridge table

**Data Quality Assessment**
- Identify and report missing values across all tables
- Fill missing `Address2` values with empty strings
- Drop merchant rows with missing ZIP codes (1,267 of 100,343 rows)
- Detect outliers using the IQR method across numeric columns in Users and Cards dimensions
- Cap outliers at IQR-derived bounds
- Bin the `Amount` column into categories (`Low`, `Medium`, `High`, `Extreme`) to handle extreme skew and support classification-based fraud detection

**Standardization**
- Scale numeric features using `scale()` for consistent ranges across dimensions
- Lowercase and trim whitespace on all text columns

### Load
Produce final fact, dimension, junk dimension, and bridge table dataframes ready for analysis or loading into a database.

## Key Design Decisions

**Amount categorization over capping** — The `Amount` column contained valid high-value transactions well above the IQR upper bound (up to $12,390 vs. a Q3 of ~$65). Capping these values would have introduced an artificial spike in the distribution. Binning into categories preserves pattern information for fraud classification while reducing noise from extreme values.

**Errors bridge table** — Since each transaction can have multiple error types, a bridge table was used to properly model the many-to-many relationship, enabling accurate error co-occurrence analysis and fraud correlation queries.

**Merchant deduplication** — The raw transactions table contained 24M rows but only ~100K unique merchants. Extracting merchants into a dedicated dimension reduced redundancy and improved query efficiency.

## Dependencies

```r
library(dplyr)
library(lubridate)
library(tidyverse)
library(ggplot2)
```

## File Structure

```
├── README.md
├── ANA515_Project_Part1_Report.pdf   # Star schema design and initial ETL overview
├── ANA515_Project_Part2_Report.pdf   # Advanced modeling and data quality report
├── ANA515_Project_Part1.Rmd          # Part 1 R script
└── ANA515_Project_Part2.Rmd          # Part 2 R script
```

> **Note:** The raw CSV files are not included in this repository due to file size. Download them directly from [Kaggle](https://www.kaggle.com/datasets/ealtman2019/credit-card-transactions/data) before running the scripts. The dataset used in this project is licensed under the Apache License 2.0.

## Author

**Anushree Das**
[LinkedIn](https://linkedin.com/in/anushree-s-das) · [GitHub](https://github.com/anushreedas) · [Medium](https://medium.com/@anushreedas.2710)
