This project uses Docker, Docker Compose, and GCP to run an ETL pipeline that stores data in PostgreSQL.

## 1. Docker Compose Setup

In the repository, you will find the `docker-compose.yaml` file, which defines the necessary services for PostgreSQL, pgAdmin, and Kestra, including the required volumes.

To start the services, run the following command:

```bash
docker-compose up -d
```

## 2. Verifying the Services
To check that everything is running:

Use this command to list the containers:

```bash
docker-compose ps
```

To view logs for a specific service (e.g., PostgreSQL):

```bash
docker-compose logs postgres
```

Access pgAdmin by navigating to http://localhost:8080 in your browser and logging in with the credentials specified in the docker-compose.yaml.

## 3. Running the ETL Process
The files `02_postgres_taxi.yaml` and `02_postgres_taxi_scheduled.yaml` are used to execute the ETL process and store data in PostgreSQL.

- `02_postgres_taxi.yaml`: Runs the ETL pipeline.
- `02_postgres_taxi_scheduled.yaml`: Schedules the ETL pipeline to run at specified intervals.

## 4. Cloud Setup (Optional)
To run the setup on Google Cloud, use the following files:

- `06_gcp_taxi.yaml`: Deploys the ETL pipeline to GCP.
- `06_gcp_taxi_scheduled.yaml`: Schedules the ETL pipeline on GCP.

## 5. BigQuery Queries for Task Questions
To answer questions 3, 4, and 5, use the following BigQuery queries:

### Question 3: Total rows in the Green Taxi data for 2020

```sql
SELECT 
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_01_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_02_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_03_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_04_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_05_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_06_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_07_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_08_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_09_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_10_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_11_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.green_tripdata_2020_12_ext`) 
AS total_filas;
```

### Question 4: Total rows in the Yellow Taxi data for 2020

```sql
SELECT 
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_01_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_02_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_03_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_04_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_05_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_06_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_07_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_08_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_09_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_10_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_11_ext`) +
  (SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2020_12_ext`) 
AS total_filas;
```

### Question 5: Total rows in the Yellow Taxi data between in March 2021

```sql
SELECT COUNT(*) FROM `projectzoomcamptaxi.zoomcamp.yellow_tripdata_2021_03_ext`
AS total_filas;
```

### Question 6: Configure the timezone to New York in a Schedule trigger

    timezone: America/New_York
