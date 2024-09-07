# Docker commands and provisioning scripts for Oracle 19c standard

## Contents [[↑](#docker-commands-and-provisioning-scripts-for-oracle-19c-standard)]

- [Docker commands and provisioning scripts for Oracle 19c standard](#docker-commands-and-provisioning-scripts-for-oracle-19c-standard)
  - [Contents \[↑\]](#contents-)
    - [Volume \[↑\]](#volume-)
    - [Image \[↑\]](#image-)
    - [Container \[↑\]](#container-)
    - [Provisioning \[↑\]](#provisioning-)
    - [Stop container \[↑\]](#stop-container-)
    - [Reference \[↑\]](#reference-)

### Volume [[↑](#docker-commands-and-provisioning-scripts-for-oracle-19c-standard)]

- Create a volume for Oracle database
  
    ```shell
    host$ docker volume create oracle-19c-data
    ```

- Make sure the volume has been created

    ```shell
    host$ docker volume inspect oracle-19c-data
    host$ docker volume ls
    ```

### Image [[↑](#docker-commands-and-provisioning-scripts-for-oracle-19c-standard)]

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

### Container [[↑](#docker-commands-and-provisioning-scripts-for-oracle-19c-standard)]

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

### Provisioning [[↑](#docker-commands-and-provisioning-scripts-for-oracle-19c-standard)]

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

### Stop container [[↑](#docker-commands-and-provisioning-scripts-for-oracle-19c-standard)]

- Stop container

    ```shell
    host$ docker stop oracle-19c-std
    ```

### Reference [[↑](#docker-commands-and-provisioning-scripts-for-oracle-19c-standard)]

- Reference:
  1. [Building Oracle Database container images](https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance#building-oracle-database-container-images)
  2. [Running Oracle Database in a container](https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance#running-oracle-database-in-a-container)
  3. [Creating an Oracle User if it doesn't already exist](https://stackoverflow.com/questions/30710990/creating-an-oracle-user-if-it-doesnt-already-exist)
