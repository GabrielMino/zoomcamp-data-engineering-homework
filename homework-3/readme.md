## Environment Setup

Before running the script, ensure that your environment has the necessary permissions and tools.

### Ensure You Have the Required Tools

1. **Python 3.8 or higher**: Make sure you have a compatible version of Python installed.

2. **Google Cloud SDK installed and authenticated**: If you haven't done so, install the SDK and authenticate your account using the following command:
   ```
   gcloud auth application-default login
    ```
3. Required libraries: Install the necessary libraries using pip:
  ```
  pip install google-cloud-storage
  ```
Google Cloud Storage Bucket Setup
1. Create a GCS bucket: If you don’t already have a Google Cloud Storage bucket, create one using the following command:

  ```
  gsutil mb -l us-central1 gs://dezoomcamp_hw3_2025
  ```
Replace dezoomcamp_hw3_2025 with your preferred bucket name.

2. Create a service account with the necessary permissions:

  1. Go to the Google Cloud Console → IAM & Admin → Service Accounts.
  2. Create a new account with the Storage Admin role.
  3. Download the JSON credentials file (gcs.json) and place it in the same directory where you will run the script.

## Overview
As follow are the responses to the Module 3 Homework questions. The data used for this assignment comes from the Yellow Taxi Trip Records dataset for January 2024 - June 2024. The dataset is stored in Google Cloud Storage (GCS) and queried using BigQuery.

## BigQuery Setup

### Creating an External Table
```sql
CREATE OR REPLACE EXTERNAL TABLE `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_external`
OPTIONS (
    format = 'PARQUET',
    uris = ['gs://dezoomcamp_hw3_2025_gm/yellow_tripdata_2024-*.parquet']
);
```

### Creating a Regular Table
```sql
CREATE OR REPLACE TABLE `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_table` AS
SELECT * FROM `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_external`;
```

---

## Questions and Answers

### Question 1: What is the count of records for the 2024 Yellow Taxi Data?
#### Query:
```sql
SELECT COUNT(*) AS total_records
FROM `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_table`;
```
#### Result:
| total_records |
|--------------|
| 20,332,093  |

---

### Question 2: Count the distinct number of PULocationIDs for both tables and estimated data read.
#### Query:
```sql
SELECT COUNT(DISTINCT PULocationID) AS distinct_pulocation
FROM `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_table`;
```
#### Result:
- **Materialized Table Data Read**: 155.12 MB
- **External Table Data Read**: 0 MB (due to caching mechanism in BigQuery)

---

### Question 3: Why are estimated bytes different when querying one column vs. two columns?
#### Queries:
```sql
SELECT PULocationID
FROM `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_table`;
```
- **Estimated Bytes Processed**: 155.12 MB

```sql
SELECT PULocationID, DOLocationID
FROM `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_table`;
```
- **Estimated Bytes Processed**: 310.24 MB

#### Explanation:
BigQuery is a **columnar** database, meaning it only reads the columns specified in the query. Since the second query retrieves two columns, it processes more data compared to the first query, which only retrieves one column.

---

### Question 4: How many records have a fare_amount of 0?
#### Query:
```sql
SELECT COUNT(*)
FROM `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_table`
WHERE fare_amount = 0;
```
#### Result:
| fare_amount_count |
|------------------|
| 8,333           |

---

### Question 5: Best strategy for an optimized table for queries filtering by `tpep_dropoff_datetime` and ordering by `VendorID`?
#### Answer:
**Partition by `tpep_dropoff_datetime` and Cluster on `VendorID`**

#### Query to create an optimized table:
```sql
CREATE OR REPLACE TABLE `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_optimized`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID
AS
SELECT * FROM `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_table`;
```

---

### Question 6: Querying distinct `VendorID` between March 1 - March 15, 2024.
#### Query (Non-Partitioned Table):
```sql
SELECT DISTINCT VendorID
FROM `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_table`
WHERE DATE(tpep_dropoff_datetime) BETWEEN '2024-03-01' AND '2024-03-15';
```
- **Bytes Processed**: 310.24 MB

#### Query (Partitioned Table):
```sql
SELECT DISTINCT VendorID
FROM `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_optimized`
WHERE DATE(tpep_dropoff_datetime) BETWEEN '2024-03-01' AND '2024-03-15';
```
- **Bytes Processed**: 26.84 MB

#### Explanation:
Using the partitioned table significantly reduces the bytes processed, making the query more efficient.

---

### Question 7: Where is the data stored for the external table?
#### Answer:
The data for the external table is stored in a **Google Cloud Storage (GCS) Bucket**.

---

### Question 8: Is it best practice to always cluster data in BigQuery?
#### Answer:
**False.**

#### Explanation:
While clustering can improve performance, it is not always necessary. Clustering is beneficial when queries frequently filter by clustered columns, but it adds overhead during data ingestion. If clustering is not aligned with query patterns, it may not provide performance benefits.

---

### Question 9 (Bonus): Estimated bytes read for `SELECT count(*)` on the materialized table.
#### Query:
```sql
SELECT COUNT(*) FROM `projectzoomcamptaxi3.yellow_taxi_dataset.yellow_taxi_table`;
```
#### Estimated Bytes Read:
**0 B**

#### Explanation:
BigQuery uses **metadata** to store the total row count of a table. When executing `SELECT COUNT(*)`, it retrieves the precomputed row count instead of scanning the entire table, resulting in **zero bytes processed**.

