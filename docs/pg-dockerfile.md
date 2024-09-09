# Setting Up PostgreSQL 14.2 with Dockerfile

This document provides instructions for setting up a PostgreSQL 14.2 database using a Dockerfile.

## Contents [[↑](#setting-up-postgresql-142-with-dockerfile)]

- [Setting Up PostgreSQL 14.2 with Dockerfile](#setting-up-postgresql-142-with-dockerfile)
  - [Contents \[↑\]](#contents-)
    - [`init-pg-user-db.sql` \[↑\]](#init-pg-user-dbsql-)
    - [`Dockerfile` \[↑\]](#dockerfile-)
    - [Build the Docker image \[↑\]](#build-the-docker-image-)
    - [Start the PostgreSQL container \[↑\]](#start-the-postgresql-container-)
    - [Connect to the PostgreSQL service \[↑\]](#connect-to-the-postgresql-service-)
    - [Stop container \[↑\]](#stop-container-)
    - [Reference \[↑\]](#reference-)

### `init-pg-user-db.sql` [[↑](#setting-up-postgresql-142-with-dockerfile)]

- Please refer to the `init-pg-user-db.sql` section in the
  [Setting Up PostgreSQL 14.2 with Docker Compose](./pg-docker-compose.md#init-pg-user-dbsql-)
  for the SQL script to create a user and a database.

### `Dockerfile` [[↑](#setting-up-postgresql-142-with-dockerfile)]

- Create a `Dockerfile` file, and add the following content.

  ```yaml
  FROM postgres:14.2

  # Copy the SQL script to create a user and a database
  COPY ./init-pg-user-db.sql /docker-entrypoint-initdb.d/
  ```

### Build the Docker image [[↑](#setting-up-postgresql-142-with-dockerfile)]

- Build the Docker image using the following command.

  ```shell
  host$ docker image build -f Dockerfile -t pg-14.2 .
  ```

### Start the PostgreSQL container [[↑](#setting-up-postgresql-142-with-dockerfile)]

- Please refer to the `Container` section in the
  [Docker commands and provisioning scripts for PostgreSQL 14.2](./pg-docker-commands.md#container-)
  to start the PostgreSQL container.

### Connect to the PostgreSQL service [[↑](#setting-up-postgresql-142-with-dockerfile)]

- Please refer to the `Connect to the PostgreSQL service` section in the
  [Docker commands and provisioning scripts for PostgreSQL 14.2](./pg-docker-commands.md#connect-to-the-postgresql-service)
  to connect to the PostgreSQL service.

### Stop container [[↑](#setting-up-postgresql-142-with-dockerfile)]

- Please refer to the `Stop container` section in the
  [Docker commands and provisioning scripts for PostgreSQL 14.2](./pg-docker-commands.md#stop-container-)
  to stop the PostgreSQL container.

### Reference [[↑](#setting-up-postgresql-142-with-dockerfile)]

- Reference:
  - [Docker Hub - Postgres](https://hub.docker.com/_/postgres)
    - start a `postgres` instance
    - `POSTGRES_PASSWORD`
    - `PGDATA`
    - `Docker Secrets`
    - `Initialization scripts`
