# Docker Compose Setup for Data Engineering with Nessie, MinIO, Spark, and Dremio

This documentation covers the services defined in the `docker-compose.yml` file, how to spin up and down the services, and where to place seed data for the MinIO and Spark containers.

## Services Overview

### Nessie (Catalog Server)
- **Image:** `projectnessie/nessie:latest`
- **Purpose:** Provides an in-memory catalog for managing data versions and branches, backed by RocksDB for persistence.
- **Ports:** 
  - Exposes API on port `19120`.
- **Volumes:**
  - The local folder `./nessie-data` is mounted to `/nessie/data` inside the container to persist RocksDB data.

### MinIO (Object Storage)
- **Image:** `minio/minio`
- **Purpose:** Object storage service, similar to AWS S3, used to store files for use in data pipelines.
- **Ports:** 
  - Exposes API on port `9000`.
  - Exposes console UI on port `9001`.
- **Volumes:**
  - The local folder `./minio-data` is mounted to `/minio-data` inside the container. Files placed here will be copied into the `seed` bucket upon initialization.
- **Healthcheck:** MinIO's health is checked via the `/minio/health/live` endpoint.

### Spark (Data Processing Engine)
- **Image:** `alexmerced/spark35nb:latest`
- **Purpose:** Provides a Spark cluster with a Jupyter Notebook interface for running data processing tasks.
- **Ports:**
  - `8080`: Spark Master Web UI.
  - `7077`: Spark Master port for job submissions.
  - `8081`: Spark Worker Web UI.
  - `4040-4045`: UI ports for individual Spark jobs.
  - `18080`: Spark History Server.
  - `8888`: Jupyter Notebook.
- **Volumes:**
  - The local folder `./notebook-seed` is mounted to `/workspace/seed-data` inside the container. This can be used to seed notebooks and data files for Spark jobs.

### Dremio (Data Lakehouse Engine)
- **Image:** `dremio/dremio-oss:latest`
- **Purpose:** Provides data analytics and querying capabilities on top of the data lake.
- **Ports:**
  - `9047`: Dremio Web UI.
  - `31010`: Dremio internal port.
  - `32010`: Dremio internal port.
  - `45678`: Dremio internal port.

## Instructions

### How to Spin Up the Services

1. **Ensure Docker and Docker Compose are installed** on your system.
2. **Navigate to the directory** containing the `docker-compose.yml` file.
3. **Place Seed Data:**
   - For MinIO, place the files to be seeded into the bucket in `./minio-data`.
   - For Spark, place any notebooks or datasets in `./notebook-seed`.
4. **Run the following command** to start all the services:
```bash
   docker-compose up -d
```
This will start all the services in the background.

### How to Spin Down the Services
To stop and remove the running containers, use the following command:

```bash
docker-compose down
```

This will stop all the services and remove the containers. Data stored in volumes (./nessie-data, ./minio-data, ./notebook-seed) will persist.

### Seed Data Locations
- **MinIO:** Files placed in the ./minio-data folder on your host will be copied into the seed bucket inside MinIO during startup.
- **Spark:** The ./notebook-seed folder on your host is mounted to /workspace/seed-data inside the Spark container. You can place Jupyter notebooks or datasets in this folder to be available in the Spark environment.

### Accessing the Services
- **Nessie API:** Access the Nessie API at http://localhost:19120.
- **MinIO Web UI:** Access the MinIO Web UI at http://localhost:9001.
- **Spark Master Web UI:** Access the Spark Master Web UI at http://localhost:8080.
- **Spark Worker Web UI:** Access the Spark Worker Web UI at http://localhost:8081.
- **Spark History Server:** Access the Spark History Server at http://localhost:18080.
- **Jupyter Notebook (Spark): Access the Jupyter Notebook interface at http://localhost:8888.
- **Dremio Web UI:** Access the Dremio Web UI at http://localhost:9047.

## Notes
Ensure that the appropriate ports (listed above) are open and not blocked by firewalls.
The services will run in a shared Docker network called intro-network, allowing them to communicate with each other.

For persistent data storage, ensure the mounted directories (./nessie-data, ./minio-data, ./notebook-seed) exist on your local machine.