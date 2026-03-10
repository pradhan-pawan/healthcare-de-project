# healthcare-de-project
End-to-end Healthcare Data Engineering Pipeline using ADF, Databricks, Snowflake, PySpark &amp; Medallion Architecture

# 🏥 Healthcare Data Engineering Pipeline
### End-to-End Azure Data Pipeline | MIMIC-III Clinical Dataset

![Azure](https://img.shields.io/badge/Azure-Data%20Factory-blue) ![Databricks](https://img.shields.io/badge/Databricks-Delta%20Lake-orange) ![Snowflake](https://img.shields.io/badge/Snowflake-Gold%20Layer-lightblue) ![Python](https://img.shields.io/badge/Python-PySpark-yellow)

---

## 📌 Project Overview

A production-grade, end-to-end data engineering pipeline built on **Microsoft Azure**, processing real-world clinical data (MIMIC-III) through a **Medallion Architecture** (Bronze → Silver → Gold) and loading it into **Snowflake** for analytics.

This project demonstrates core data engineering skills including:
- Cloud data ingestion and orchestration with **Azure Data Factory**
- Data transformation and feature engineering with **Databricks + PySpark**
- Delta Lake storage on **ADLS Gen2**
- Analytical data loading into **Snowflake**
- End-to-end pipeline automation

---

## 🏗️ Architecture

```
Raw CSV Files (MIMIC-III)
        │
        ▼
┌─────────────────┐
│   Bronze Layer  │  ADF Pipeline → CSV to Parquet (ADLS Gen2)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Silver Layer  │  Databricks → Clean, Cast, Deduplicate → Delta
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Gold Layer    │  Databricks → Aggregations, KPIs → Delta
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Snowflake     │  Spark Snowflake Connector → GOLD schema
└─────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Cloud Platform | Microsoft Azure |
| Orchestration | Azure Data Factory (ADF) |
| Storage | Azure Data Lake Storage Gen2 (ADLS) |
| Compute | Azure Databricks (Apache Spark) |
| File Format | Parquet → Delta Lake |
| Data Warehouse | Snowflake |
| Language | Python (PySpark) |
| Version Control | Git + GitHub |
| Secrets Management | Databricks Secret Scope |

---

## 📁 Project Structure

```
healthcare-de-project/
│
├── notebooks/
│   ├── 01_bronze_to_silver.ipynb   # Silver transformation notebook
│   ├── 02_silver_to_gold.ipynb     # Gold aggregation notebook
│   └── 03_gold_to_snowflake.ipynb  # Snowflake loading notebook
│
├── adf/
│   ├── PL_Bronze_Ingestion.json    # ADF Bronze pipeline
│   └── PL_Master_Healthcare.json   # ADF Master pipeline
│
├── raw_data/                        # MIMIC-III source CSVs
│
└── README.md
```

---

## 📊 Dataset — MIMIC-III (Clinical)

| File | Rows | Description |
|---|---|---|
| `PATIENTS.csv` | 100 | Patient demographics |
| `ADMISSIONS.csv` | 129 | Hospital admissions |
| `DIAGNOSES_ICD.csv` | 1,761 | ICD-9 diagnosis codes |
| `PRESCRIPTIONS.csv` | 7,642 | Medication prescriptions |

---

## 🔄 Pipeline Stages

### Stage 1 — Bronze Layer (ADF)
- **Pipeline:** `PL_Bronze_Ingestion`
- Reads 5 raw CSVs from ADLS `bronze/raw_data/`
- Converts to **Parquet** format using ForEach + Copy Activity
- Writes to `bronze/processed/`

### Stage 2 — Silver Layer (Databricks)
- **Notebook:** `01_bronze_to_silver`
- Reads Bronze Parquet files
- Applies: type casting, deduplication, null handling
- Engineers features: `is_deceased`, `age_at_death`, `LOS`, `is_hospital_death`, `is_primary_diagnosis`, `drug_days`
- Writes 4 Delta tables to `silver/`

### Stage 3 — Gold Layer (Databricks)
- **Notebook:** `02_silver_to_gold`
- Builds 4 analytical tables:
  - `patient_summary` — 100 rows (total admissions, avg LOS, hospital deaths per patient)
  - `readmission_analysis` — 3 rows (9.24% emergency readmission rate)
  - `diagnosis_trends` — 622 rows (top ICD-9 codes by frequency)
  - `medication_analysis` — 792 rows (top drugs by prescription count)
- Writes as Delta format to `gold/`

### Stage 4 — Snowflake Loading (Databricks)
- **Notebook:** `03_gold_to_snowflake`
- Uses Snowflake Spark Connector
- Loads all 4 Gold tables into `HEALTHCARE_DB.GOLD` schema
- Credentials managed via **Databricks Secret Scope**

### Stage 5 — Master Pipeline (ADF)
- **Pipeline:** `PL_Master_Healthcare`
- Orchestrates full end-to-end run:
```
Bronze Ingestion → Silver Notebook → Gold Notebook → Snowflake Notebook
```

---

## 📈 Key Metrics (Gold Layer Results)

| Metric | Value |
|---|---|
| Total Patients | 100 |
| Total Admissions | 129 |
| Emergency Readmission Rate | 9.24% |
| Unique ICD-9 Diagnosis Codes | 622 |
| Unique Medications Tracked | 792 |

---

## ⚙️ Setup & Configuration

### Prerequisites
- Azure Subscription
- Databricks Workspace
- Snowflake Account
- Azure Data Factory

### Secrets Configuration
```bash
# Store ADLS key in Databricks secret scope
databricks secrets put-secret healthcare-scope adls-access-key --string-value "<your-key>"

# Store Snowflake password
databricks secrets put-secret healthcare-scope snowflake-password --string-value "<your-password>"
```

### Snowflake Setup
```sql
CREATE DATABASE HEALTHCARE_DB;
CREATE WAREHOUSE HEALTHCARE_WH WITH WAREHOUSE_SIZE = 'X-SMALL' AUTO_SUSPEND = 60;
CREATE SCHEMA HEALTHCARE_DB.GOLD;
```

---

## 🚀 Running the Pipeline

### Full End-to-End Run
1. Go to **ADF Studio** → Pipelines → `PL_Master_Healthcare`
2. Click **Debug** or **Add Trigger** → **Trigger Now**
3. Monitor in ADF Monitor tab

### Expected Runtime
| Activity | Duration |
|---|---|
| Bronze Ingestion | ~55s |
| Silver Notebook | ~1m 37s |
| Gold Notebook | ~51s |
| Snowflake Load | ~1m 2s |
| **Total** | **~4 minutes** |

---

## 🔐 Security

- ADLS access keys stored in **Databricks Secret Scope** (never hardcoded)
- Snowflake credentials stored in **Databricks Secret Scope**
- Role-based access: `ACCOUNTADMIN` for Snowflake
- ADF uses **Managed Identity** where possible

---

## 👨‍💻 Author

**Pawan** | Data Engineering Project  
GitHub: [pradhan-pawan/healthcare-de-project](https://github.com/pradhan-pawan/healthcare-de-project)

---

*Built with ❤️ using Azure, Databricks, and Snowflake*