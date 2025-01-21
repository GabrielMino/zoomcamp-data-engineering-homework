
# Project Setup Instructions

This project involves setting up a PostgreSQL database and a PgAdmin container using Docker to load data from two CSV URLs into the database. Follow these instructions to get started:

### Ensure Volume Creation
Before starting, make sure the Docker volume is created, as it is required for the PostgreSQL database container to persist its data.

### `docker-compose.yml` File
Below is the configuration for your Docker Compose setup:

```yaml
services:
  pgdatabase:
    image: postgres:13
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=root
      - POSTGRES_DB=ny_taxi
    volumes:
      - homework-1_postgres_data:/var/lib/postgresql/data:rw
    ports:
      - "5432:5432"
  pgadmin:
    image: dpage/pgadmin4
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@admin.com
      - PGADMIN_DEFAULT_PASSWORD=root
    ports:
      - "8080:80"

volumes:
  homework-1_postgres_data:
    external: true
```

### Running Docker Containers
1. **Start Containers**: Run the following command to bring up the Docker containers defined in the `docker-compose.yml` file:

   ```bash
   docker-compose up -d
   ```

   This command starts the containers in detached mode.

2. **Check Running Containers**: Once the containers are running, you can verify their status by executing:

   ```bash
   docker ps
   ```

   This will display the active containers and their respective ports. It’s important to verify that both the PostgreSQL and PgAdmin containers are up and running.

### Build the Docker Image
Next, build the Docker image using the provided Dockerfile:

```bash
docker build -t taxi_ingest:v003 .
```

This creates the `taxi_ingest:v003` image, which is used to ingest the data into the PostgreSQL database.

### Define URLs for Data Files
You will need to define the URLs for the data you want to ingest into the database. Set the following environment variables:

```bash
URL_GREEN="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz"
URL_ZONES="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv"
```

These URLs point to the green taxi trip data and taxi zone lookup data respectively.

### Running the Data Ingestion
Run the following commands to ingest the data into the PostgreSQL database:

1. **Ingest Green Taxi Data**:

   ```bash
   docker run -it    --network homework-1_default    taxi_ingest:v003    --user=root    --password=root    --host=homework-1-pgdatabase-1    --port=5432    --db=ny_taxi    --table_name=green_taxi_trips    --url=$URL_GREEN
   ```

2. **Ingest Taxi Zone Data**:

   ```bash
   docker run -it    --network homework-1_default    taxi_ingest:v003    --user=root    --password=root    --host=homework-1-pgdatabase-1    --port=5432    --db=ny_taxi    --table_name=taxi_zone_trips    --url=$URL_ZONES
   ```

### Explanation of Changes
- **Network**: We need to add the `--network homework-1_default` option because the `taxi_ingest:v003` container must communicate with the PostgreSQL container. By default, Docker containers are isolated, so specifying the network allows them to communicate.
  
- **Host**: The host must be set to `homework-1-pgdatabase-1` because this is the name of the PostgreSQL container within the Docker Compose setup. Without this, the `taxi_ingest:v003` container would not be able to locate the database server.

### Conclusion
After running these commands, the green taxi data and the taxi zone lookup data will be loaded into the `ny_taxi` database under their respective table names (`green_taxi_trips` and `taxi_zone_trips`). You can then access the database via PgAdmin on port `8080`.
