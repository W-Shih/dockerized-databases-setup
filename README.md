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
    - [Oracle 19c standard with docker container and volume](#oracle-19c-standard-with-docker-container-and-volume)

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
    one time container:
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
    or
    restart container always:
      ```shell
      host$ docker container run
      --restart=always
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

  - One time container

    ```shell
    host$ docker container run --name mssql-2019 --hostname mssql-2019 --rm -p 1433:1433 -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Test123456!!!!!" --shm-size 2g -v mssql-2019-data:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2019-latest
    ```

    or

  - Restart container always

    ```shell
    host$ docker container run --name mssql-2019 --hostname mssql-2019 --restart=always -p 1433:1433 -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Test123456!!!!!" --shm-size 2g -v mssql-2019-data:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2019-latest
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

### Oracle 19c standard with docker container and volume

- Create a volume for Oracle database
  
    ```shell
    host$ docker volume create oracle-19c-data
    ```

- Make sure the volume has been created

    ```shell
    host$ docker volume inspect oracle-19c-data
    host$ docker volume ls
    ```

- Build Oracle 19c standard image by the official oracle dockerfile
  - To build this image, your host should have bash environment locally. `Git Bash` or `WSL` is sufficient.
  - Go to the [Github - oracle/docker-images](https://github.com/oracle/docker-images)
  - Have a proper local directory ready and clone the repo by

    ```shell
    host-bash$ git clone https://github.com/oracle/docker-images.git
    ```

  - Download `LINUX.X64_193000_grid_home.zip` at [Oracle Database 19c (19.3) for Linux x86-64](https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html)

  - Put the above `LINUX.X64_193000_grid_home.zip` at `your-oracle-docker-images-repo/OracleDatabase/SingleInstance/dockerfiles/19.3.0`

  - Start building the image

    ```shell
    host-bash$ cd your-oracle-docker-images-repo/OracleDatabase/SingleInstance/dockerfiles
    host-bash$ ./buildContainerImage.sh -v 19.3.0 -s
    ```

  - Check the image built properly, should see the image `oracle/database:19.3.0-se2` by

    ```shell
    host-bash$ docker image ls
    ```

- Start Oracle container with the volume

  - The following ports are default ports exposed: 1521 (Oracle Listener), 5500 (OEM Express), 2484 (TCPS Listener Port if TCPS is enabled).
  - My these ports are occupied, so use 15210 (Oracle Listener), 55000 (OEM Express) instead.
  - Note that the highest TCP port number is **65535**.
  - Note that ORACLE_SID  must be **alphanumeric**.

    <!-- For readability
    one time container:
      ```shell
      host$ docker container run
      --rm
      --name oracle-19c-std
      -p 15210:1521 -p 55000:5500
      -e ORACLE_SID=ORCL19cCDB
      -e ORACLE_PDB=url_service_demo
      -e ORACLE_PWD="Test123456!!!!!"
      -e INIT_SGA_SIZE=4096
      -e INIT_PGA_SIZE=1024
      -e ORACLE_EDITION=standard
      -v oracle-19c-data:/opt/oracle/oradata
      -d oracle/database:19.3.0-se2
    ```
    or
    restart container always:
      ```shell
      host$ docker container run
      --restart=always
      --name oracle-19c-std
      -p 15210:1521 -p 55000:5500
      -e ORACLE_SID=ORCL19cCDB
      -e ORACLE_PDB=url_service_demo
      -e ORACLE_PWD="Test123456!!!!!"
      -e INIT_SGA_SIZE=4096
      -e INIT_PGA_SIZE=1024
      -e ORACLE_EDITION=standard
      -v oracle-19c-data:/opt/oracle/oradata
      -d oracle/database:19.3.0-se2
      ```
    -->

  - One time container

    ```shell
    host$ docker container run --name oracle-19c-std --rm -p 15210:1521 -p 55000:5500 -e ORACLE_SID=ORCL19cCDB -e ORACLE_PWD="Test123456!!!!!" -e INIT_SGA_SIZE=4096 -e INIT_PGA_SIZE=1024 -v oracle-19c-data:/opt/oracle/oradata -d oracle/database:19.3.0-se2
    ```

    or

  - Restart container always

    ```shell
    host$ docker container run --name oracle-19c-std --restart=always -p 15210:1521 -p 55000:5500 -e ORACLE_SID=ORCL19cCDB -e ORACLE_PWD="Test123456!!!!!" -e INIT_SGA_SIZE=4096 -e INIT_PGA_SIZE=1024 -v oracle-19c-data:/opt/oracle/oradata -d oracle/database:19.3.0-se2
    ```

    <!-- Test oracle-19c-std-wo-pdb
    ```shell
    host$ docker container run --name oracle-19c-std-wo-pdb --rm -p 15210:1521 -p 55000:5500 -e ORACLE_SID=ORCL19cCDB -e ORACLE_PWD="Test123456!!!!!" -e INIT_SGA_SIZE=4096 -e INIT_PGA_SIZE=1024 -d oracle/database:19.3.0-se2
    ```
    -->

    <!-- Test oracle-19c-std-w-pdb
     ```shell
    host$ docker container run --name oracle-19c-std-w-pdb --rm -p 15211:1521 -p 55001:5500 -e ORACLE_SID=ORCL19cCDB -e ORACLE_PDB=url_service_demo -e ORACLE_PWD="Test123456!!!!!" -e INIT_SGA_SIZE=4096 -e INIT_PGA_SIZE=1024 -d oracle/database:19.3.0-se2
    ```
    -->

- Check Oracle container is running properly

  - Note that Oracle container initialization takes a while (~30 mins) to finish.
  - Use the following command to see the status. `health: starting` and `healthy` refers to initializing and ready, respectively.
    - Note that the status might be showing `unhealthy` during container initialization. Please see the container's log to have a better judgement.

    ```shell
    host$ docker container logs oracle-19c-std
    host$ docker container ls
    ```

- Create a database (pdb) and an admin user for the service

    ```shell
    host$ docker exec -it oracle-19c-std /bin/bash
    # use sqlplus to connect to ORACLE_SID (cdb) as sysdba with password 
    # created in docker container run
    container$ sqlplus sys/'Test123456!!!!!'@localhost:1521/ORCL19cCDB as sysdba;
    ```

  - Note that a cdb only allows certain number of pdb to be created. In my case, it is 5. Use the following command to check the max number of pdbs allowed to be created.

    ```sql
    -- check the max number of pdbs allowed to be created
    orale> SELECT name FROM v$pdbs;
    ```

    If the max number of pdbs allowed to be created is reached, use the following command to drop a pdb.

    ```sql
    -- close the pdb
    orale> ALTER PLUGGABLE DATABASE url_service_demo CLOSE IMMEDIATE;
    -- drop a pdb
    orale> DROP PLUGGABLE DATABASE url_service_demo INCLUDING DATAFILES;
    ```
  
  - Simple version if the database and the admin user do NOT exist

    ```sql
    -- create the pdb and user
    oracle> CREATE PLUGGABLE DATABASE url_service_demo
                ADMIN USER url_service_dba IDENTIFIED BY "Test123456!!!!!"
                ROLES = (dba)
                DEFAULT TABLESPACE url_service
                    DATAFILE '/opt/oracle/oradata/ORCL19CCDB/url_service_demo/url_service01.dbf' SIZE 100M 
                AUTOEXTEND ON
                FILE_NAME_CONVERT = ('/opt/oracle/oradata/ORCL19CCDB/pdbseed/',
                                     '/opt/oracle/oradata/ORCL19CCDB/url_service_demo/')
                STORAGE (MAXSIZE 2G)
                PATH_PREFIX = '/opt/oracle/oradata/ORCL19CCDB/url_service_demo/';
    
    -- open the pdb and save
    -- without save state, the pdb will be closed after container restart
    orale> ALTER PLUGGABLE DATABASE url_service_demo OPEN;
    orale> ALTER PLUGGABLE DATABASE url_service_demo SAVE STATE;
    
    -- make sure the service is active
    oracle> SELECT name, network_name FROM v$active_services ORDER BY 1;
    
    -- connect to the service as sysdba
    oracle> ALTER SESSION SET CONTAINER=url_service_demo;
    
    -- grant privilege to user
    oracle> ALTER USER url_service_dba QUOTA UNLIMITED ON url_service CONTAINER=CURRENT;
    oracle> GRANT CONNECT TO url_service_dba CONTAINER=CURRENT;
    oracle> GRANT CREATE SESSION TO url_service_dba CONTAINER=CURRENT;
    oracle> GRANT RESOURCE TO url_service_dba CONTAINER=CURRENT;
    ```

  - To be compatible if the database or the admin user DOES exist
  
    ```sql
    -------------------------------------------------------------------------------------------------------------------
    -- create the pdb and user
    set serveroutput on format wrapped;
    DECLARE
        sql_statement varchar2(512);
        pdb_exists EXCEPTION;
        PRAGMA EXCEPTION_INIT(pdb_exists, -65012);
    BEGIN
        sql_statement := 'CREATE PLUGGABLE DATABASE url_service_demo ' ||
            'ADMIN USER url_service_dba ' ||
            'IDENTIFIED BY "Test123456!!!!!" ' ||
            'ROLES = (dba) ' ||
            'DEFAULT TABLESPACE url_service ' ||
            'DATAFILE ''/opt/oracle/oradata/ORCL19CCDB/url_service_demo/url_service01.dbf'' SIZE 100M ' || 
            'AUTOEXTEND ON FILE_NAME_CONVERT = (' ||
                '''/opt/oracle/oradata/ORCL19CCDB/pdbseed'',' ||
                '''/opt/oracle/oradata/ORCL19CCDB/url_service_demo''' ||
            ') ' ||
            'STORAGE (MAXSIZE 2G) ' ||
            'PATH_PREFIX = ''/opt/oracle/oradata/ORCL19CCDB/url_service_demo/''
            ';
        EXECUTE IMMEDIATE sql_statement;
        dbms_output.put_line('OK: ' || sql_statement);
    EXCEPTION
        WHEN pdb_exists THEN
            dbms_output.put_line('WARN: ' || sql_statement);
            dbms_output.put_line('Pluggable database name already exists. Skipping.');
        WHEN OTHERS THEN
            dbms_output.put_line('FAIL: ' || sql_statement);
            RAISE;
    END;
    /
    ```

    ```sql
    -------------------------------------------------------------------------------------------------------------------
    -- open the pdb and save
    -- without save state, the pdb will be closed after container restart
    ALTER PLUGGABLE DATABASE url_service_demo OPEN FORCE;
    ALTER PLUGGABLE DATABASE url_service_demo SAVE STATE;

    -- make sure the database is open (READ & WRITE) instead of MOUNTED
    SELECT name, open_mode from v$pdbs;

    -- make sure the service is active
    SELECT name, network_name FROM v$active_services ORDER BY 1;
    ```

    ```sql
    -------------------------------------------------------------------------------------------------------------------
    -- connect to the pdb as sysdba
    ALTER SESSION SET CONTAINER=url_service_demo;
    ```

    ```sql
    -------------------------------------------------------------------------------------------------------------------
    -- grant privilege to user
    ALTER USER url_service_dba QUOTA UNLIMITED ON url_service CONTAINER=CURRENT;
    GRANT CONNECT TO url_service_dba CONTAINER=CURRENT;
    GRANT CREATE SESSION TO url_service_dba CONTAINER=CURRENT;
    GRANT RESOURCE TO url_service_dba CONTAINER=CURRENT;
    ```

- Stop container

    ```shell
    host$ docker stop oracle-19c-std
    ```

- Reference:
  1. [Building Oracle Database container images](https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance#building-oracle-database-container-images)
  2. [Running Oracle Database in a container](https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance#running-oracle-database-in-a-container)
  3. [Creating an Oracle User if it doesn't already exist](https://stackoverflow.com/questions/30710990/creating-an-oracle-user-if-it-doesnt-already-exist)
