# IoT Sensor Data Lakehouse with AWS Glue, Kinesis & Delta Lake  
*A Production-Grade Cloud Data Engineering Pipeline for IoT Predictive Maintenance*

This project implements a **scalable, cloud-native IoT data pipeline** designed for predictive maintenance use cases. It simulates industrial sensor telemetry, ingests streaming data via **AWS Kinesis**, persists raw payloads in **Amazon S3**, performs **batch ETL using PySpark on AWS Glue**, and writes structured outputs to **Delta Lake tables**. The pipeline flags anomalies and prepares versioned datasets for downstream analytics and machine learning.

Orchestration is handled by **Apache Airflow**, with hourly scheduling to support near real-time operational insights.

---

## Architecture Overview

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
         │ AWS Glue Job (PySpark)     │
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
- Simulates telemetry for industrial equipment:
  - `temperature`, `pressure`, `vibration`, `equipment_id`, `timestamp`
- Streams data to **Kinesis** for ingestion.
- Persists raw batches in **S3** for ETL.
- Scheduled hourly via Airflow DAG.

### Cloud-Orchestrated ETL with PySpark and Glue
- Glue job reads raw JSON from S3.
- Aggregates metrics per equipment (`avg_temperature`, `avg_pressure`, `avg_vibration`).
- Applies rule-based anomaly detection.
- Writes partitioned **Delta Lake tables** for efficient querying.
- Supports incremental append mode for versioned history.

### Predictive Maintenance Ready
- Flags anomalies based on domain thresholds.
- Delta tables serve as clean, queryable input for ML pipelines.
- Enables proactive failure detection and maintenance scheduling.

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Orchestration** | Apache Airflow | DAG scheduling & workflow management |
| **Streaming** | AWS Kinesis Data Stream | Real-time sensor ingestion |
| **Data Lake / Storage** | AWS S3 | Durable raw and processed storage |
| **Batch ETL** | AWS Glue (PySpark) | Aggregation, anomaly detection, Delta write |
| **Data Format** | Delta Lake | Versioned, append-only, queryable tables |
| **Language** | Python | Sensor simulation & ETL logic |
| **Libraries** | PySpark, awsglue, boto3 | AWS SDK integration & transformation logic |

---

## Output Tables (Delta Lake)

### **iot_data_delta**
Stored in S3 under `processed/iot_data_delta/`.  
Partitioned by `equipment_id` for optimized access.

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

> Data is incrementally appended each DAG run, creating a versioned history for analytics and ML.

---

## Workflow Summary

1. **Airflow DAG triggers** hourly:
   - `generate_sensor_data` → simulates sensor readings.
   - `verify_s3_files` → validates raw ingestion.
   - `s3_to_delta_batch_job` → transforms and writes to Delta Lake.
2. **ETL logic**:
   - Timestamp normalization
   - Equipment-level aggregation
   - Anomaly flagging
3. **Delta Lake** stores clean, queryable tables.
4. **Predictive Maintenance**:
   - ML models and dashboards consume Delta tables.
   - Enables early detection of equipment failure patterns.

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

This pipeline delivers a **robust, production-grade IoT lakehouse architecture**, combining streaming ingestion, scalable ETL, and predictive analytics readiness.
