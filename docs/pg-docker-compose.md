# Setting Up PostgreSQL 14.2 with Docker Compose

This guide provides steps to set up a PostgreSQL 14.2 database using Docker Compose. It includes a
`docker-compose.yml` file to define the PostgreSQL service and a provisioning script to create a database and user.

## Contents [[↑](#setting-up-postgresql-142-with-docker-compose)]

- [Setting Up PostgreSQL 14.2 with Docker Compose](#setting-up-postgresql-142-with-docker-compose)
  - [Contents \[↑\]](#contents-)
    - [`init-pg-user-db.sql` \[↑\]](#init-pg-user-dbsql-)
    - [`docker-compose.yml` \[↑\]](#docker-composeyml-)
    - [Start the PostgreSQL service using Docker Compose \[↑\]](#start-the-postgresql-service-using-docker-compose-)
    - [Connect to the PostgreSQL service \[↑\]](#connect-to-the-postgresql-service-)
    - [Useful Docker Compose Commands \[↑\]](#useful-docker-compose-commands-)
    - [References \[↑\]](#references-)

### `init-pg-user-db.sql` [[↑](#setting-up-postgresql-142-with-docker-compose)]

- Create a directory for the PostgreSQL service.

  ```shell
  host$ mkdir pg-14.2
  host$ cd pg-14.2
  ```

- Create a `init-pg-user-db.sql` file, and add the following content.

  ```sql
  CREATE USER url_service_dba WITH PASSWORD 'Test123456!!!!!';
  CREATE DATABASE url_service_demo;
  GRANT ALL PRIVILEGES ON DATABASE url_service_demo TO url_service_dba;
  ```

  Note that there's no need to ensure the SQL scripts (`init-pg-user-db.sql` or more) are re-executable even if the
  database or the admin user DOES exist as long as the volume is not deleted.

  When the volume is not deleted, the SQL scripts will only run the first time the container is created and the volume
  is empty. This is because the `/docker-entrypoint-initdb.d/` mechanism in the official PostgreSQL Docker image is
  specifically designed for database initialization. It only runs the scripts when the container starts for the first
  time and no existing data is detected.

  - First container creation: When you run `docker-compose up` for the first time and a new volume is created, the SQL
    scripts will execute to initialize the database (creating tables, users, data, etc.).
  - Container restart: When you restart the container or start a new container with an existing volume, the SQL scripts
    will not run again because the data already exists in the volume. The PostgreSQL container detects that the
    database has already been initialized and skips these steps.
  - Recreating the container: Even if you delete the container, as long as the volume is not deleted and the data still
    exists, the SQL scripts will not run again, as the volume retains the previous initialization.

  Therefore, If you want to re-run the SQL scripts with the same volume name, you need delete the volume and recreate
  it. This will ensure that the SQL scripts run again when the container starts.

  ```shell
  host$ docker-compose down
  host$ docker volume rm <volume_name>
  ```

  Then, run `docker-compose up` to recreate the container with the volume and re-run the SQL scripts.

  ```shell
  host$ docker-compose up -d
  ```

  This will re-run the SQL scripts and re-initialize the user and database.

### `docker-compose.yml` [[↑](#setting-up-postgresql-142-with-docker-compose)]

- Create a `docker-compose.yml` file, and add the following content.

  ```yaml
  version: '3.8'

  services:
    postgres:
      image: postgres:14.2
      environment:
        # POSTGRES_PASSWORD is the password for the default `postgres` user, this env variable is required to run 
        # the PostgreSQL container.
        # 
        # Note: 
        #   It is not recommended to set the password this way because it is visible to everyone who has access to 
        #   the `docker-compose.yml` file. 
        #   It is not recommended to use a `.env` file to store the password and use it in the `docker-compose.yml` 
        #   file either because this env variable will be set in the container and can be seen by running the command 
        #     `docker inspect <container_id/container_name>` or 
        #     `docker exec -it <container_id/container_name> env`.
        #
        # TODO:
        #   It is recommended to use the docker secrets or docker config to store the password securely.
        POSTGRES_PASSWORD: "Test123456!!!!!"
      ports:
        - "5432:5432"
      volumes:
        - pg-14.2-data:/var/lib/postgresql/data
        - ./init-pg-user-db.sql:/docker-entrypoint-initdb.d/init-pg-user-db.sql
      restart: always

  volumes:
    pg-14.2-data:
  ```

### Start the PostgreSQL service using Docker Compose [[↑](#setting-up-postgresql-142-with-docker-compose)]

- Start the PostgreSQL service using Docker Compose.

  ```shell
  host$ docker-compose up -d
  ```

- Check that the PostgreSQL service is running properly.

  ```shell
  host$ docker container ls
  ```

### Connect to the PostgreSQL service [[↑](#setting-up-postgresql-142-with-docker-compose)]

- Please refer to the `Connect to the PostgreSQL service` section in the
  [Docker commands and provisioning scripts for PostgreSQL 14.2](./pg-docker-commands.md#connect-to-the-postgresql-service)
  to connect to the PostgreSQL service.

### Useful Docker Compose Commands [[↑](#setting-up-postgresql-142-with-docker-compose)]

- Stop the PostgreSQL service using Docker Compose.

  ```shell
  host$ docker-compose stop
  ```

- Start the PostgreSQL service using Docker Compose.

  ```shell
  host$ docker-compose start
  ```

- Stop and remove the PostgreSQL service container using Docker Compose.

  ```shell
  host$ docker-compose down
  ```

- For more docker-compose commands, refer to
  
  ```shell
  host$ docker-compose --help
  ```

### References [[↑](#setting-up-postgresql-142-with-docker-compose)]

- Reference:
  - [Docker Hub - Postgres](https://hub.docker.com/_/postgres)
    - Example `docker-compose.yml` for `postgres`
    - `POSTGRES_PASSWORD`
    - `PGDATA`
    - `Docker Secrets`
    - `Initialization scripts`
