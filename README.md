# IoT Sensor Data Lakehouse with AWS Glue, Kinesis & Delta Lake  
*A Production-Grade Cloud Data Engineering Pipeline for IoT Predictive Maintenance*

This project is a fully automated **IoT data pipeline** that simulates sensor readings from industrial equipments, stores the raw data in **S3**, performs **batch ETL with AWS Glue**, writes to **Delta Lake tables**, and prepares anomaly-flagged datasets for **predictive maintenance analytics**.

The pipeline is orchestrated end-to-end using **Apache Airflow** and scheduled to run hourly for streaming analytics.

---


## Architecture Overview

### **Architecture**
![Architecture Flow](Assets/architect.png)

     ┌───────────────────────────┐
     │  IoT Sensor Simulation    │
     │ (PythonOperator in Airflow) │
     └─────────────┬─────────────┘
                   │
                   ▼
     ┌───────────────────────────┐
     │  Kinesis Stream            │
     │  IoTSensorStream           │
     └─────────────┬─────────────┘
                   │
                   ▼
     ┌───────────────────────────┐
     │  S3 Raw Bucket             │
     │  s3://my-iot-lakehouse/raw │
     └─────────────┬─────────────┘
                   │
                   ▼
     ┌───────────────────────────┐
     │ AWS Glue Batch Job         │
     │  s3_to_delta_batch.py      │
     └─────────────┬─────────────┘
                   │
                   ▼
     ┌───────────────────────────┐
     │ Delta Lake Tables          │
     │  s3://my-iot-lakehouse/processed/iot_data_delta │
     │  (Aggregated + Anomaly Flags) │
     └─────────────┬─────────────┘
                   │
                   ▼
     ┌───────────────────────────┐
     │ Predictive Maintenance     │
     │ Analytics / ML Models      │
     └───────────────────────────┘


---

## Key Features

### IoT Sensor Data Simulation
- Generates **randomized sensor readings** for equipment:
  - `temperature`, `pressure`, `vibration`, `equipment_id`, `timestamp`
- Sends data to **Kinesis stream** for streaming purposes.
- Backs up each batch to **S3 raw storage** for batch ETL.
- **Scheduled hourly** via Airflow DAG.

### Cloud-Orchestrated ETL with Glue
- Glue job `S3_to_DeltaLake_Batch_Job` reads raw JSON from S3.
- Aggregates data per equipment (`avg_temperature`, `avg_pressure`, `avg_vibration`).
- Detects anomalies using thresholds.
- Writes **Delta Lake tables** partitioned by `equipment_id`.
- Supports incremental updates (append mode).

### Predictive Maintenance Ready
- Anomalies in temperature, pressure, vibration are flagged automatically.
- Delta Lake tables serve as **clean, versioned, queryable input** for ML models.
- Enables proactive maintenance alerts.

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Orchestration** | Apache Airflow | DAG scheduling & workflow management |
| **Streaming** | AWS Kinesis Data Stream | Sensor data ingestion |
| **Data Lake / Storage** | AWS S3 | Raw & processed sensor storage |
| **Batch ETL** | AWS Glue (PySpark) | Read, aggregate, anomaly detection, write Delta |
| **Data Format** | Delta Lake | Versioned, append-only, queryable tables |
| **Language** | Python | ETL & sensor simulation logic |
| **Libraries** | PySpark, awsglue, boto3 | ETL, Delta, AWS SDK integration |

---

## Output Tables (Delta Lake)

### **iot_data_delta**
- Stored in S3 under `processed/iot_data_delta/`
- Schema:

| Column | Description |
|--------|-------------|
| equipment_id | Unique ID of each machine |
| avg_temperature | Average temperature in batch |
| avg_pressure | Average pressure in batch |
| avg_vibration | Average vibration in batch |
| temp_anomaly | True if temperature > 80°C |
| pressure_anomaly | True if pressure > 4 bar |
| vibration_anomaly | True if vibration > 4.5 |
| processed_at | ETL processing timestamp |

> Data is **incrementally appended** each DAG run, creating a **versioned history** for analytics and predictive models.

---

## Workflow Summary

1. **Airflow DAG triggers** hourly:
   - `generate_sensor_data` → simulates sensor readings.
   - `verify_s3_files` → ensures raw files exist in S3.
   - `s3_to_delta_batch_job` → Glue job transforms raw data into Delta Lake.
2. **Data transformation**:
   - Convert timestamp
   - Compute aggregates per equipment
   - Flag anomalies
3. **Delta Lake** stores clean, queryable tables for ML or BI.
4. **Predictive Maintenance**:
   - ML models or analytics dashboards query Delta tables.
   - Detects patterns and predicts potential equipment failure.

---

## Project Screenshots 

### **1. Airflow DAG**
![Airflow DAG](Assets/airflow2.png)


![Airflow DAG](Assets/airflow.png)

### **2. AWS S3 Buckets**
![S3](Assets/S3.png)
![S3](Assets/s3bucket.png)

### **3. AWS Glue**
![](Assets/glue.png)
![](Assets/scripts.png)

### **4. AWS Airflow Environment**
![](Assets/environment1.png)
![](Assets/environment2.png)

### **5. AWS Kinesis**
![](Assets/kinesis1.png)
![](Assets/kinesis2.png)

### **6. Delta Lake Tables**
- Versioned tables with anomaly flags for each equipment.
![](Assets/delta1.png)
![](Assets/delta2.png)



---

This setup ensures a **robust, production-grade IoT ETL pipeline**, with hourly data ingestion, automated anomaly detection, and predictive maintenance readiness.
