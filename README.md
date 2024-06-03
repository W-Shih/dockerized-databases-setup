<!--===================================================================================================================
File description:
    This file is to introduce how to set databases (pg, mssql, oracle) up for a web-app-service by docker.

====================================================================================================================-->

# Dockerized databases setup

This repository provides guides and scripts to set up PostgreSQL, MS-SQL, and Oracle databases using Docker.

It includes steps for creating and managing database instances, ensuring persistence with Docker volumes, and configuring database users and schemas. Detailed commands and instructions are provided for smooth database initialization and management.

# Contents

- [Dockerized databases setup](#dockerized-databases-setup)
- [Contents](#contents)
  - [Set up Databases](#set-up-databases)
    - [PostgreSQL 14.2 with docker container and volume](#postgresql-142-with-docker-container-and-volume)
    - [MS-SQL 2019 with docker container and volume](#ms-sql-2019-with-docker-container-and-volume)

## Set up Databases

### PostgreSQL 14.2 with docker container and volume

- Create a volume for PostgreSQL
  
    ```shell
    host$ docker volume create pg-14.2-data
    ```

- Make sure the volume has been created

    ```shell
    host$ docker volume inspect pg-14.2-data
    host$ docker volume ls
    ```

- Start pg container with the volume

    <!-- For readability
    ```shell
    host$ docker container run
    --rm
    --name pg-14.2
    -p 5432:5432
    -e POSTGRES_PASSWORD="Test123456!!!!!"
    -v pg-14.2-data:/var/lib/postgresql/data
    -d postgres:14.2
    ```
    -->

    ```shell
    host$ docker container run --name pg-14.2 --rm -p 5432:5432 -e POSTGRES_PASSWORD="Test123456!!!!!" -v pg-14.2-data:/var/lib/postgresql/data -d postgres:14.2
    ```

- Check pg container is running properly

    ```shell
    host$ docker container ls
    ```

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

- Stop container

    ```shell
    host$ docker stop pg-14.2
    ```

- Reference:
  1. [Docker Hub - Postgres](https://hub.docker.com/_/postgres)
  2. [Simulate CREATE DATABASE IF NOT EXISTS for PostgreSQL](https://stackoverflow.com/questions/18389124/simulate-create-database-if-not-exists-for-postgresql)

### MS-SQL 2019 with docker container and volume

- Create a volume for MS SQL Server
  
    ```shell
    host$ docker volume create mssql-2019-data
    ```

- Make sure the volume has been created

    ```shell
    host$ docker volume inspect mssql-2019-data
    host$ docker volume ls
    ```

- Start MS SQL Server container with the volume

    <!-- For readability
    ```shell
    host$ docker container run
    --rm
    --name mssql-2019
    --hostname mssql-2019
    --shm-size 2g
    -p 1433:1433
    -e "ACCEPT_EULA=Y"
    -e "MSSQL_SA_PASSWORD=Test123456!!!!!"
    -v mssql-2019-data:/var/opt/mssql
    -d mcr.microsoft.com/mssql/server:2019-latest
    ```
    -->

    ```shell
    host$ docker container run --name mssql-2019 --hostname mssql-2019 --rm -p 1433:1433 -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Test123456!!!!!" --shm-size 2g -v mssql-2019-data:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2019-latest
    ```

- Check MS SQL Server container is running properly

    ```shell
    host$ docker container ls
    ```

- Create a database and an admin user for the service

    ```shell
    host$ docker exec -it mssql-2019 /bin/bash
    container# /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'Test123456!!!!!'
    ```

  - Simple version if the database and the admin user do NOT exist

    ```sql
    mssql> CREATE DATABASE url_service_demo;
    mssql> GO
    mssql> USE url_service_demo;
    mssql> GO
    mssql> CREATE LOGIN url_service_dba WITH PASSWORD = 'Test123456!!!!!';
    mssql> GO
    mssql> CREATE USER url_service_dba FOR LOGIN url_service_dba;
    mssql> GO
    mssql> ALTER ROLE db_owner ADD MEMBER url_service_dba;
    mssql> GO
    ```

  - To be compatible if the database or the admin user DOES exist

    ```sql
    -------------------------------------------------------------------------------------------------------------------
    -- create database if it not exists
    DECLARE
        @db_name VARCHAR(32) = 'url_service_demo',
        @sql_template VARCHAR(MAX) = 'CREATE DATABASE {DB_NAME}',
        @sql_statement VARCHAR(MAX);
    BEGIN TRY
        SET @sql_statement = REPLACE(@sql_template, '{DB_NAME}', @db_name);
        EXECUTE (@sql_statement);
    END TRY
    BEGIN CATCH
        IF ERROR_NUMBER() = 1801
            PRINT N'Database name already exists. Skipping.';
        ELSE
            THROW;
    END CATCH
    GO
    ```

    or

    ```sql
    DECLARE
        @db_name VARCHAR(32) = 'url_service_demo',
        @sql_template VARCHAR(MAX) = 'CREATE DATABASE {DB_NAME}',
        @sql_statement VARCHAR(MAX);
    IF EXISTS (
        SELECT name FROM master.dbo.sysdatabases WHERE name=@db_name
    )
        BEGIN
            PRINT N'Database name already exists. Skipping.';
        END
    ELSE
        BEGIN
            SET @sql_statement = REPLACE(@sql_template, '{DB_NAME}', @db_name);
            EXECUTE (@sql_statement);
        END
    GO
    ```

    ```sql
    -------------------------------------------------------------------------------------------------------------------
    -- use the created database
    USE url_service_demo;
    GO
    ```

    ```sql
    -------------------------------------------------------------------------------------------------------------------
    -- create login user if it not exists
    DECLARE
        @login_name VARCHAR(32) = 'url_service_dba',
        @login_password VARCHAR(128) = '''Test123456!!!!!''';
    DECLARE 
        @sql_statement VARCHAR(MAX) = 'CREATE LOGIN ' + @login_name + ' WITH PASSWORD =' + @user_password;
    IF SUSER_ID(@login_name) IS NULL
        EXECUTE (@sql_statement);
    ELSE
        PRINT N'Login user name already exists. Skipping.';
    GO

    -- create admin user if it not exists
    DECLARE
        @login_name VARCHAR(32) = 'url_service_dba';
    DECLARE
        @user_name VARCHAR(32) = @login_name;
    DECLARE 
        @sql_statement VARCHAR(MAX) = 'CREATE USER ' + @user_name + ' FOR LOGIN ' + @login_name;
    IF DATABASE_PRINCIPAL_ID(@user_name) IS NULL
        EXECUTE (@sql_statement);
    ELSE
        PRINT N'Database user name already exists. Skipping.';
    GO

    -------------------------------------------------------------------------------------------------------------------
    -- grant all privileges
    ALTER ROLE db_owner ADD MEMBER url_service_dba;
    GO
    ```

- Stop container

    ```shell
    host$ docker stop mssql-2019
    ```

- Reference:
  1. [Docker Hub - Microsoft SQL Server](https://hub.docker.com/_/microsoft-mssql-server)
  2. [Run SQL Server Linux container images with Docker](https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-linux-ver15&preserve-view=true&pivots=cs1-cmd#pullandrun2019)
  3. [MS SQL Server - Use data volume containers](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-docker-container-configure?view=sql-server-ver16&pivots=cs1-powershell#use-data-volume-containers)
  4. [SQL Create Database: A How-To Guide](https://www.databasestar.com/sql-create-database/#Check_if_Database_Is_Created)
  5. [Does MSSQL support an equivalent form of the MySQL statement `CREATE DATABASE IF NOT EXISTS {db}`](https://stackoverflow.com/questions/64103200/does-mssql-support-an-equivalent-form-of-the-mysql-statement-create-database-if)
  6. [Check if a user exists in a SQL Server database](https://dba.stackexchange.com/questions/125886/check-if-a-user-exists-in-a-sql-server-database)
