# Week 6 – Azure Data Factory (ADF) ETL Project

In this project, we designed an ETL pipeline using **Azure Data Factory (ADF)** to extract and load data from multiple sources. The pipeline connects to:

- An **on-premises SQL Server**
- An **SFTP server**

The data extracted from these sources is then ingested into:

- **Azure SQL Database**
- **Azure Data Lake Storage (ADLS)**

To optimize the workflow, we implemented **incremental loading** so that only new or updated records are processed during each run. Additionally, we configured a trigger to automate pipeline execution on the **last Saturday of every month**.

---

## 📂 Dataset Used

**NYC Restaurant Inspection Data**

This dataset was chosen due to its monthly updates and substantial size, making it ideal for demonstrating incremental loading techniques.

---

## 🔁 Pipelines Overview

### ✅ Pipeline 1: `Run_Pipeline_OnPrem_To_SQLDB`
Transfers data from the on-premises SQL Server to Azure SQL Database.

### ✅ Pipeline 2: `Run_Pipeline_SFTP_To_ADLS`
Moves files from the SFTP server into Azure Data Lake Storage.

### 🧩 Master Pipeline
Combines both pipelines and includes a monthly trigger to run the entire ETL process on a schedule.

---

## ⚙️ Setup: Data Sources & Targets

### 📌 On-Premises SQL Server
- Configured using **SQL Server Management Studio (SSMS)**
- Loaded with the **NYC Restaurant Inspection Data**

### 📌 SFTP Server
- Set up via **WinSCP**
- Contains **3 CSV files**, each representing data split by year from the main dataset

---

### 📥 Load Targets

#### 🗄️ Azure SQL Database
- Deployed under the `week6` resource group
- Receives cleaned and transformed data from the SQL Server source

#### 🌐 Azure Data Lake Storage (ADLS)
- Contains two folders:
  - `SFTP-DATA`
  - `Restaurant_Inspection`

---

### 🚀 Pipeline 1: On-Premise SQL Server → Azure SQL Database
<p align="center">
  <img src="./images/1st_Pipeline.png" alt="1stPipeline" />
</p>

### 1. Linked Services
 - On-Premise SQL Server (via Self-Hosted Integration Runtime)
 - Azure SQL Database

### 2. Datasets
 - Source: sqlservertable1
 - Sink: azuresqltable1

## 🔄 Incremental Load Logic

I applied **watermarking** to detect and process only new or changed records. This approach uses a combination of **Lookup** and **Filter** activities.

### 🔎 Step 1: Lookup Watermark
```sql
SELECT MAX([INSPECTION DATE]) AS last_watermark
FROM dbo.NYC_Restaurant
```

### 🧹 Step 2: Filter New Records
```sql
SELECT *
FROM dbo.DOHMH_New_York_City_Restaurant_$
WHERE [INSPECTION DATE] > '@{activity('Lookup_Watermark').output.firstRow.last_watermark}'
```

### 🔁 Step 3: Update Watermark Table
```sql
UPDATE watermark_table
SET last_updated_time = (
    SELECT MAX([INSPECTION DATE])
    FROM dbo.NYC_Restaurant
)
WHERE table_name = 'NYC_NYC_Restaurant';
```
### 📦 Pipeline 2: SFTP Server → Azure Data Lake Storage

### 1. Linked Services
 - SFTP Server (connected via SHIR)
 - ADLS Gen2

### 2. Datasets
 - Source: CSV files from SFTP
 - Sink: ADLS folders

### 3. Copy Activity
 - Source: SFTP files
 - Destination: ADLS folders
---

### 🧠 Master Pipeline + Trigger
 - Combines 1st_Pipeline and 2nd_Pipeline
 - Adds a trigger to run on the last Saturday of every month at 7 AM

## 📚 Summary

This project demonstrates how to build scalable, automated ETL solutions using Azure Data Factory, complete with real-time scheduling and incremental data processing across multiple systems.
