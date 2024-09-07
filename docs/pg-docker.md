# Docker commands and provisioning scripts for PostgreSQL 14.2

## Contents [[↑](#docker-commands-and-provisioning-scripts-for-postgresql-142)]

- [Docker commands and provisioning scripts for PostgreSQL 14.2](#docker-commands-and-provisioning-scripts-for-postgresql-142)
  - [Contents \[↑\]](#contents-)
    - [Volume \[↑\]](#volume-)
    - [Container \[↑\]](#container-)
    - [Provisioning \[↑\]](#provisioning-)
    - [Stop container \[↑\]](#stop-container-)
    - [Reference \[↑\]](#reference-)

### Volume [[↑](#docker-commands-and-provisioning-scripts-for-postgresql-142)]

- Create a volume for PostgreSQL
  
    ```shell
    host$ docker volume create pg-14.2-data
    ```

- Make sure the volume has been created

    ```shell
    host$ docker volume inspect pg-14.2-data
    host$ docker volume ls
    ```

### Container [[↑](#docker-commands-and-provisioning-scripts-for-postgresql-142)]

- Start pg container with the volume

    <!-- For readability
    one time container:
      ```shell
      host$ docker container run
      --rm
      --name pg-14.2
      -p 5432:5432
      -e POSTGRES_PASSWORD="Test123456!!!!!"
      -v pg-14.2-data:/var/lib/postgresql/data
      -d postgres:14.2
      ```
    or
    restart container always:
      ```shell
      host$ docker container run
      --restart=always
      --name pg-14.2
      -p 5432:5432
      -e POSTGRES_PASSWORD="Test123456!!!!!"
      -v pg-14.2-data:/var/lib/postgresql/data
      -d postgres:14.2
      ```
    -->

  - One time container

    ```shell
    host$ docker container run --name pg-14.2 --rm -p 5432:5432 -e POSTGRES_PASSWORD="Test123456!!!!!" -v pg-14.2-data:/var/lib/postgresql/data -d postgres:14.2
    ```

    or

  - Restart container always

    ```shell
    host$ docker container run --name pg-14.2 --restart=always -p 5432:5432 -e POSTGRES_PASSWORD="Test123456!!!!!" -v pg-14.2-data:/var/lib/postgresql/data -d postgres:14.2
    ```

- Check pg container is running properly

    ```shell
    host$ docker container ls
    ```

### Provisioning [[↑](#docker-commands-and-provisioning-scripts-for-postgresql-142)]

- Create a database and an admin user for the service

    ```shell
    host$ docker exec -it pg-14.2 /bin/bash
    container# psql -U postgres
    ```

  - Simple version if the database and the admin user do NOT exist

    ```sql
    pg# create user url_service_dba with password 'Test123456!!!!!';
    pg# create database url_service_demo;
    pg# grant all privileges on database url_service_demo to url_service_dba;
    ```

  - To be compatible if the database or the admin user DOES exist

    ```sql
    CREATE EXTENSION IF NOT EXISTS dblink;
    ```

    ```sql
    -------------------------------------------------------------------------------------------------------------------
    -- create admin user if it not exists
    DO
    $do$
    DECLARE
        user_name TEXT := 'url_service_dba';
        user_password TEXT := '''Test123456!!!!!''';
        sql_statement varchar;
    BEGIN
        IF EXISTS (SELECT FROM pg_user WHERE  usename=user_name) THEN
            RAISE NOTICE 'Role name already exists. Skipping.';
        ELSE
            BEGIN   -- nested block
                sql_statement := 'CREATE ROLE ' || user_name || ' LOGIN PASSWORD ' || user_password;
                PERFORM dblink_exec(
                    'dbname=' || current_database(),  -- dblink creates another connection to current db
                    sql_statement
                );
            EXCEPTION
                WHEN duplicate_object THEN
                    RAISE NOTICE 'Role name was just created by a concurrent transaction. Skipping.';
            END;
        END IF;
    END
    $do$
    ```

    ```sql
    -------------------------------------------------------------------------------------------------------------------
    -- create database if it not exists
    SELECT 'CREATE DATABASE url_service_demo' WHERE NOT EXISTS (SELECT FROM pg_database WHERE datname='url_service_demo')\gexec
    ```

    or

    ```sql
    DO
    $do$
    DECLARE
        db_name TEXT := 'url_service_demo';
        sql_statement varchar;
    BEGIN
        IF EXISTS (SELECT FROM pg_database WHERE datname=db_name) THEN
            RAISE NOTICE 'Database name already exists. Skipping.';  -- optional
        ELSE
            sql_statement := 'CREATE DATABASE ' || db_name;
            PERFORM dblink_exec(
                'dbname=' || current_database(),  -- dblink creates another connection to current db
                sql_statement
            );
    END IF;
    END
    $do$;  
    ```

    ```sql
    -------------------------------------------------------------------------------------------------------------------
    -- grant all privileges
    grant all privileges on database url_service_demo to url_service_dba;
    ```

### Stop container [[↑](#docker-commands-and-provisioning-scripts-for-postgresql-142)]

- Stop container

    ```shell
    host$ docker stop pg-14.2
    ```

### Reference [[↑](#docker-commands-and-provisioning-scripts-for-postgresql-142)]

- Reference:
  1. [Docker Hub - Postgres](https://hub.docker.com/_/postgres)
  2. [Simulate CREATE DATABASE IF NOT EXISTS for PostgreSQL](https://stackoverflow.com/questions/18389124/simulate-create-database-if-not-exists-for-postgresql)
