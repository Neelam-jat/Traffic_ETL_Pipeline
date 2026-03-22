# 🚦 NHTSA Traffic Data: ETL Pipeline & Star Schema Design

![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![Pandas](https://img.shields.io/badge/Library-Pandas-orange)
![Architecture](https://img.shields.io/badge/Architecture-Star_Schema-purple)
![Domain](https://img.shields.io/badge/Data-NHTSA_FARS-green)

## 📖 Executive Summary

Raw government data is often stored in massive, "flat" files that are difficult to query efficiently. This project demonstrates a **Data Engineering** workflow using the **NHTSA FARS (Fatality Analysis Reporting System)** dataset.

The goal was to build an **ETL (Extract, Transform, Load)** pipeline that converts raw crash records into a **Star Schema**. This architecture reduces data redundancy and optimizes the dataset for analytical queries regarding weather conditions, time of day, and casualty metrics.

---

## 🏗️ Technical Architecture: The Star Schema

To optimize storage and query performance, I moved away from a single flat-file approach and implemented a Relational Model.

### 1. The Fact Table (`fact_crash`)
*   **Role:** The central table containing quantitative metrics and foreign keys.
*   **Grain:** One row per accident case.
*   **Key Columns:** `CASENUM` (Primary Key), `VE_TOTAL` (Vehicles Involved), `NUM_INJ` (Injuries), `Env_ID` (FK), `Time_ID` (FK).

### 2. Dimension: Environment (`dim_environment`)
*   **Role:** Stores the context regarding road conditions.
*   **Why?** In the raw data, weather codes (e.g., `WEATHER1`) and light conditions (`LGT_COND`) are repeated thousands of times. By creating a dimension table, we store each unique combination once, assigning it a unique `Env_ID`.

### 3. Dimension: Time (`dim_time`)
*   **Role:** Handles temporal data.
*   **Transformation:** Aggregated `YEAR`, `MONTH`, `DAY_WEEK`, and `HOUR` into a unified `Time_ID`, allowing for efficient drilling down into specific timeframes without scanning the entire dataset.

---

## ⚙️ The ETL Process

### 1. Extract
*   Ingested the `acc_16.csv` (FARS 2016) dataset using Python/Pandas.
*   Performed initial schema validation to ensure column integrity.

### 2. Transform (Normalization)
*   **Deduplication:** Identified unique permutations of Weather and Light conditions.
*   **Surrogate Keys:** Generated synthetic IDs (`Env_ID`, `Time_ID`) to link Dimensions to the Fact table.
*   **Merging:** Mapped the original raw data against these new keys to create the lightweight `fact_crash` table.

### 3. Load
*   Exported the normalized tables as separate CSV entities, mimicking the process of loading tables into a SQL Data Warehouse (e.g., Snowflake or PostgreSQL).

---

## 💻 Code Snippet: Logic for Dimension Creation

```python
# Creating the Environment Dimension to normalize Weather/Light data
dim_environment = df[['WEATHER1', 'LGT_COND']].drop_duplicates().reset_index(drop=True)
dim_environment['Env_ID'] = dim_environment.index + 1 

# Mapping back to the Fact Table
fact_table = df.merge(dim_environment, on=['WEATHER1', 'LGT_COND'], how='left')
