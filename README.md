# Lottery Data Warehouse & ETL Project

## Overview

This project contains a PostgreSQL-based Data Warehouse (DWH) and ETL pipeline implementation for lottery sales and transaction data.

The repository includes:

* Source schema setup scripts
* 3NF (Third Normal Form) data warehouse layer
* Data Mart (DM) dimensional model
* ETL procedures and transformation logic
* Incremental loading procedures
* Fact table partitioning
* Date dimension generation
* Test scripts

The implementation uses PostgreSQL PL/pgSQL stored procedures, functions, sequences, schemas, and partitioning.

---

# Project Structure

```text
Demo Final Project Final/
│
├── DDL_DML _sa_src.sql
├── DDL_bl_cl.sql
├── DDL_bl_3nf.sql
├── DDL_bl_dm.sql
├── DML_bl_cl.sql
├── DML_bl_3nf.sql
├── DML_bl_dm.sql
├── ETL_Pipeline_Procedure.sql
├── Fact_table_partitioning.sql
├── Task4_Create_Date_Hierarchy_Date.sql
├── Test_Partitions.sql
└── Introduction_to_DWH_and_ETL_Business_Template_*.docx
```

---

# Technology Stack

* PostgreSQL
* PL/pgSQL
* pgAdmin
* PostgreSQL FDW (`file_fdw`)

---

# Database Architecture

The solution uses multiple schemas:

| Schema             | Purpose                                    |
| ------------------ | ------------------------------------------ |
| `sa_final_draw`    | Source area for draw-based lottery data    |
| `sa_final_scratch` | Source area for scratch-based lottery data |
| `bl_cl`            | Cleansing and staging layer                |
| `bl_3nf`           | Normalized enterprise warehouse layer      |
| `bl_dm`            | Dimensional model / star schema            |

---

# Prerequisites

Before running the project, ensure the following are installed:

## 1. PostgreSQL

Recommended version:

* PostgreSQL 13+

## 2. pgAdmin

Install pgAdmin to manage and execute SQL scripts.

## 3. CSV Source Files

The project expects CSV files to exist locally.

The comments in the scripts indicate that:

* A local `csv` folder should exist
* PostgreSQL must have read permissions to the files

Example:

```text
C:\csv\
```

---

# Initial Database Setup

## Step 1 — Create Database

Create a PostgreSQL database.

Example:

```sql
CREATE DATABASE lottery;
```

Connect to the database before executing the remaining scripts.

---

## Step 2 — Enable Required Extension

The project uses PostgreSQL Foreign Data Wrapper for file access.

```sql
CREATE EXTENSION IF NOT EXISTS file_fdw;
```

This is also included in the setup script.

---

# Recommended Execution Order

The scripts should be executed in the following order.

---

## 1. Source Layer Setup

### File

```text
DDL_DML _sa_src.sql
```

### Purpose

* Creates source schemas
* Creates base warehouse schemas
* Enables extensions
* Prepares source ingestion environment

### Run

```sql
\i 'DDL_DML _sa_src.sql'
```

---

## 2. Cleansing Layer Objects

### File

```text
DDL_bl_cl.sql
```

### Purpose

Creates:

* Metadata tables
* Working tables
* Cleansing layer structures

### Run

```sql
CALL bl_cl.create_bl_cl_objects();
```

---

## 3. 3NF Layer Objects

### File

```text
DDL_bl_3nf.sql
```

### Purpose

Creates:

* Sequences
* 3NF warehouse tables
* Core enterprise structures

### Run

```sql
CALL bl_cl.create_bl_3nf_sequences();
```

Additional procedures/functions inside the script may also need execution depending on your environment.

---

## 4. Dimensional Model Objects

### File

```text
DDL_bl_dm.sql
```

### Purpose

Creates:

* Surrogate key sequences
* Dimension tables
* Fact tables
* Data mart structures

### Run

```sql
CALL bl_cl.create_bl_dm_sequences();
```

---

## 5. Metadata and Cleansing Procedures

### File

```text
DML_bl_cl.sql
```

### Purpose

Loads:

* Metadata mappings
* Cleansing procedures
* Staging logic

### Example Run

```sql
CALL bl_cl.sp_insert_meta_game_types();
```

---

## 6. 3NF ETL Procedures

### File

```text
DML_bl_3nf.sql
```

### Purpose

Creates:

* Transformation functions
* Merge logic
* Incremental ETL procedures

---

## 7. Dimensional Model ETL

### File

```text
DML_bl_dm.sql
```

### Purpose

Creates:

* Dimension upsert procedures
* Fact loading procedures
* SCD handling logic

Example procedure:

```sql
CALL bl_cl.sp_upsert_dim_game_numbers(CURRENT_DATE);
```

---

## 8. Main ETL Pipeline

### File

```text
ETL_Pipeline_Procedure.sql
```

### Purpose

Defines:

* Batch loading procedures
* End-to-end ETL orchestration
* Incremental processing

### Main Procedure

```sql
CALL bl_cl.p_load_all_ce_data(CURRENT_DATE);
```

---

## 9. Date Dimension

### File

```text
Task4_Create_Date_Hierarchy_Date.sql
```

### Purpose

Creates and populates:

* `bl_dm.dim_date`

Includes:

* Fiscal hierarchy
* Calendar hierarchy
* Month naming attributes

---

## 10. Fact Table Partitioning

### File

```text
Fact_table_partitioning.sql
```

### Purpose

Implements:

* Incremental loading improvements
* Fact table partitioning
* Optimized ETL logic

---

## 11. Testing

### File

```text
Test_Partitions.sql
```

### Purpose

Provides:

* Repeatability tests
* Partition validation
* ETL execution tests

Example:

```sql
CALL bl_cl.sp_run_batch_etl_by_day();
```

---

# ETL Workflow

The overall pipeline follows this architecture:

```text
CSV Files
   ↓
Source Area (sa_*)
   ↓
Cleansing Layer (bl_cl)
   ↓
3NF Warehouse (bl_3nf)
   ↓
Dimensional Model (bl_dm)
   ↓
Reports / Analytics
```

---

# Key Features

## Incremental Loading

Several procedures support incremental loads using dates.

Example:

```sql
CALL bl_cl.p_load_all_ce_data('2025-01-01');
```

---

## Slowly Changing Dimensions (SCD)

The dimensional layer appears to support SCD logic for customer dimensions.

Example procedure:

```sql
sp_upsert_dim_customers_scd
```

---

## Fact Table Partitioning

Partitioning scripts are included to improve:

* Query performance
* ETL scalability
* Data management

---

# Example Full Execution Flow

```sql
-- Step 1
\i 'DDL_DML _sa_src.sql'

-- Step 2
\i 'DDL_bl_cl.sql'
CALL bl_cl.create_bl_cl_objects();

-- Step 3
\i 'DDL_bl_3nf.sql'
CALL bl_cl.create_bl_3nf_sequences();

-- Step 4
\i 'DDL_bl_dm.sql'
CALL bl_cl.create_bl_dm_sequences();

-- Step 5
\i 'DML_bl_cl.sql'

-- Step 6
\i 'DML_bl_3nf.sql'

-- Step 7
\i 'DML_bl_dm.sql'

-- Step 8
\i 'ETL_Pipeline_Procedure.sql'

-- Step 9
\i 'Task4_Create_Date_Hierarchy_Date.sql'

-- Step 10
\i 'Fact_table_partitioning.sql'

-- Step 11
CALL bl_cl.sp_run_batch_etl_by_day();
```

---

# Troubleshooting

## Permission Errors on CSV Files

Ensure PostgreSQL has permission to read the CSV directory.

Windows example:

```text
C:\csv\
```

Grant read access to the PostgreSQL service account.

---

## Extension Errors

If `file_fdw` fails:

```sql
CREATE EXTENSION file_fdw;
```

You may need superuser privileges.

---

## Schema Already Exists

Most scripts use:

```sql
IF NOT EXISTS
```

However, for a clean rebuild you may manually drop schemas:

```sql
DROP SCHEMA bl_dm CASCADE;
```

Use carefully.

---

# Suggested Improvements

Potential enhancements for production use:

* Add transaction handling
* Add ETL logging tables
* Add audit framework
* Add automated scheduling
* Add Docker deployment
* Add dbt or orchestration tooling
* Add unit tests
* Add CI/CD pipeline

---

# Author Notes

This project demonstrates:

* Data warehouse architecture
* ETL orchestration
* PostgreSQL procedural programming
* Dimensional modeling
* Partitioning strategies
* Incremental data loading

It is suitable for:

* Academic projects
* ETL demonstrations
* PostgreSQL DWH learning
* BI pipeline prototyping

---

# License

Add a license if the project will be shared publicly.
