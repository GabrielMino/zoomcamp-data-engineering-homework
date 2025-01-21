
# Module 1 Homework: Docker & SQL

In this homework we'll prepare the environment and practice Docker and SQL.

## Question 1. Understanding docker first run
Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash.
**What's the version of pip in the image?**

To find out the version of pip in the python:3.12.8 image, you can run the following commands inside the Docker container:


docker run -it python:3.12.8 bash
pip --version

**Answer:** 24.3.1

---

## Question 2. Understanding Docker networking and docker-compose
Given the following docker-compose.yaml, what is the hostname and port that pgadmin should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```
Given the docker-compose.yaml file, we have the following details:

The PostgreSQL service is named db, and it uses port 5433 on the host machine, which maps to 5432 inside the container.
The pgAdmin service will connect to PostgreSQL using the service name db as the hostname. Since port 5432 inside the container is mapped to 5433 on the host, pgAdmin should connect to the db service on port 5433.
Thus, the hostname and port for pgAdmin to connect to PostgreSQL will be:

**Answer:** db:5433

---

## Question 3. Trip Segmentation Count
During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, respectively, happened:

- Up to 1 mile
- In between 1 (exclusive) and 3 miles (inclusive),
- In between 3 (exclusive) and 7 miles (inclusive),
- In between 7 (exclusive) and 10 miles (inclusive),
- Over 10 miles

Query: 

```sql
SELECT 
    COUNT(*) AS trip_count,
    CASE
        WHEN trip_distance <= 1 THEN '1- Up to 1 mile'
        WHEN trip_distance > 1 AND trip_distance <= 3 THEN '2- Between 1 and 3 miles'
        WHEN trip_distance > 3 AND trip_distance <= 7 THEN '3- Between 3 and 7 miles'
        WHEN trip_distance > 7 AND trip_distance <= 10 THEN '4- Between 7 and 10 miles'
        WHEN trip_distance > 10 THEN '5- Over 10 miles'
    END AS distance_range
FROM public.green_taxi_trips
WHERE lpep_pickup_datetime >= '2019-10-01' 
  AND lpep_pickup_datetime < '2019-11-01' 
  AND lpep_dropoff_datetime < '2019-11-01'
GROUP BY distance_range
ORDER BY distance_range;
```

**Answer:** 104,802; 198,924; 109,603; 27,678; 35,189

---

## Question 4. Longest trip for each day
Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.

Query:

```sql
SELECT DATE(lpep_pickup_datetime) AS pickup_day,
MAX(trip_distance) AS longest_distance
FROM green_taxi_trips
GROUP BY DATE(lpep_pickup_datetime)
ORDER BY longest_distance DESC
LIMIT 1;
```

**Answer:** 2019-10-31

---

## Question 5. Three biggest pickup zones
Which were the top pickup locations with over 13,000 in total_amount (across all trips) for 2019-10-18?

Query: 

```sql
SELECT 
    loc."Zone" AS pickup_zone,
    trips.total_amount_sum
FROM (
    SELECT 
        "PULocationID" AS pickup_location,
        SUM(total_amount) AS total_amount_sum
    FROM green_taxi_trips trips
    WHERE DATE(lpep_pickup_datetime) = '2019-10-18'
    GROUP BY pickup_location
    HAVING SUM(total_amount) > 13000
    ORDER BY total_amount_sum DESC
	LIMIT 3
) trips
JOIN taxi_zone_trips loc
ON trips.pickup_location = loc."LocationID";
```

**Answer:** East Harlem North, East Harlem South, Morningside Heights

---

## Question 6. Largest tip
For the passengers picked up in October 2019 in the zone name "East Harlem North" which was the drop off zone that had the largest tip?

Query:

```sql
SELECT 
    loc."Zone",
    MAX(t.tip_amount) AS max_tip
FROM 
    green_taxi_trips t
JOIN taxi_zone_trips loc 
ON t."DOLocationID" = loc."LocationID"  -- Join with the zones table to get the drop-off zone names
WHERE 
    t.lpep_pickup_datetime >= '2019-10-01' 
    AND t.lpep_pickup_datetime < '2019-11-01'
    AND t."PULocationID" = (SELECT "LocationID" FROM taxi_zone_trips WHERE taxi_zone_trips."Zone" = 'East Harlem North') -- Ensure the pickup zone is "East Harlem North"
GROUP BY 
    loc."Zone"
ORDER BY max_tip DESC
LIMIT 1;
```

**Answer:** JFK Airport

---
## Question 7. Terraform Workflow
Which of the following sequences, respectively, describes the workflow for:

- Downloading the provider plugins and setting up backend,
- Generating proposed changes and auto-executing the plan
- Remove all resources managed by terraform

**Answer:** terraform init, terraform apply -auto-approve, terraform destroy
