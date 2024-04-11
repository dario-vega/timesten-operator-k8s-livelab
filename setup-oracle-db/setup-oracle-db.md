# Set Up the Oracle Database to Cache Data

## Introduction

Before you can use TimesTen Cache, you must create the following users in your Oracle database:

- A cache administration user. This user creates and maintains Oracle Database objects that store information about the cache environment.
This user also enforces predefined behaviors of cache group types.

- One or more schema users who owns Oracle Database tables that are cached in a TimesTen database.


Estimated Time: 20 minutes

### Objective
* Setup a database to use with TimesTen Cache

### Prerequisites
* You have executed Lab 2: Install the Oracle Database Kubernetes Operator

## Task 1: Create a ConfigMap Object for the Database Network configuration

This section creates the sample ConfigMap. This ConfigMap contains the tnsnames configuration of the Oracle DB.

On your Linux development host:

1. From the directory of your choice, create an empty subdirectory for the metadata files.
This example creates the `cm_tns` subdirectory.

    ```
    <copy>mkdir -p cm_tns</copy>
    ```
2. Navigate to the ConfigMap directory.

    ```
    <copy>cd cm_tns</copy>
    ```
3. Create the tnsnames.ora file in this ConfigMap directory.

    ```
    <copy>vi tnsnames.ora</copy>
    ```
    ```
    <copy>
    cache_high =  (description= (retry_count=20)(retry_delay=3)(address=(protocol=tcps)(port=1521)(host=l4xr4q5i.adb.us-ashburn-1.oraclecloud.com))(connect_data=(service_name=rddainsuh6u1okc_cache_high.adb.oraclecloud.com))(security=(ssl_server_dn_match=no)))
    </copy>
    ```

4. Create the sqlnet.ora file in this ConfigMap directory.
    ```
    <copy>vi sqlnet.ora</copy>
    ```
    ```
    <copy>
    NAME.DIRECTORY_PATH= {TNSNAMES, EZCONNECT, HOSTNAME}
    SQLNET.EXPIRE_TIME = 10
    SSL_VERSION = 1.2
    </copy>
    ```

5. Create the ConfigMap.

    ```  
    <copy>
    cd ..
    kubectl create configmap tns --from-file=cm_tns
    </copy>

    configmap/tns created
    ```
    You successfully created and deployed the `tns` ConfigMap.

6. Use the kubectl describe command to verify the contents of the ConfigMap.
    ```
    <copy>kubectl describe configmap tns</copy>

    Name:         tns
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Data
    ====
    sqlnet.ora:
    ----
    NAME.DIRECTORY_PATH= {TNSNAMES, EZCONNECT, HOSTNAME}
    SQLNET.EXPIRE_TIME = 10
    SSL_VERSION = 1.2

    tnsnames.ora:
    ----
    cache_high =  (description= (retry_count=20)(retry_delay=3)(address=(protocol=tcps)(port=1521)(host=l4xr4q5i.adb.us-ashburn-1.oraclecloud.com))(connect_data=(service_name=rddainsuh6u1okc_cache_high.adb.oraclecloud.com))(security=(ssl_server_dn_match=no)))


    BinaryData
    ====

    Events:  <none>

    ```

## Task 2: Create the TimesTen cache administration Oracle Database Users

1. Use the kubectl create command to create the TimesTenClassic object from the contents of the YAML file.

    ```
    <copy>kubectl create -f client.yaml</copy>
    pod/client created created
    ```

2. Create a shell from which you can access your Oracle Database

    ```
    <copy>
    kubectl exec -it client -c client -- /bin/bash
    </copy>
    ```

    ```
    <copy>
    export TNS_ADMIN=/tns
    stty erase ^H
    VER=$(ls -1rt /timesten/installations)
    echo "define oraclescriptspath=/timesten/installations/$VER/oraclescripts" > /tmp/oraclescriptspath.sql
    </copy>
    ```



3. use SQL*Plus to connect to the Oracle Database as the sys or ADMIN user

    For non-Autonomous DB
    ```
    <copy>
    sqlplus sys@cache_high as sysdba
    </copy>
    ```
    For Autonomous DB
    ```
    <copy>
    sqlplus admin@cache_high
    </copy>
    ```

4. Use the SQL*Plus session to create a default tablespace to store the TimesTen Cache management objects.

    For non-Autonomous DB
    ```
    <copy>
    CREATE TABLESPACE cachetablespace2 DATAFILE 'datatt2.dbf' SIZE 100M;
    </copy>
    ```
    For Autonomous DB
    ```
    ERROR at line 1:
    ORA-01031: insufficient privileges
    ```
    This error indicates that you are not allowed to run the SQL command in Autonomous Database.
    More [here](https://docs.oracle.com/en-us/iaas/autonomous-database-serverless/doc/autonomous-sql-commands.html)

5. Use the SQL*Plus session to create the cache administration user

    For non-Autonomous DB
    ```
    <copy>
    CREATE USER cacheuser2 IDENTIFIED BY Oraclepwd##2020
       DEFAULT TABLESPACE cachetablespace2
       QUOTA UNLIMITED ON cachetablespace2;

    </copy>
    ```
    For Autonomous DB
    ```
    <copy>
    CREATE USER cacheuser2 IDENTIFIED BY Oraclepwd##2020
       QUOTA UNLIMITED ON DATA;
    </copy>
    ```

6. Use the SQL*Plus session to Grant Privileges to the Cache Administration User

    ```
    <copy>
    @/tmp/oraclescriptspath.sql
    define oraclescriptspath
    </copy>
    ```
    ```
    <copy>      
    @&oraclescriptspath./grantCacheAdminPrivileges.sql "cacheuser2"
    SET FEED ON
    </copy>
    ```

## Task 3: Create the Oracle Database Tables to Be Cached

1. Use the SQL*Plus session to create the schema user. Grant this schema user the minimum privileges
required to create tables in the Oracle Database to be cached in your TimesTen database
    ```
    <copy>
    CREATE USER oratt IDENTIFIED BY Oraclepwd##2020 QUOTA UNLIMITED ON DATA;
    GRANT CREATE SESSION, RESOURCE TO oratt;

    </copy>
    ```

2. Use the SQL*Plus session to create the tables
    ```
    <copy>
    CREATE TABLE oratt.readtab (keyval NUMBER NOT NULL PRIMARY KEY,
       str VARCHAR2(32));

    CREATE TABLE oratt.writetab (pk NUMBER NOT NULL PRIMARY KEY,
       attr VARCHAR2(40));

    </copy>
    ```

3. Use the SQL*Plus session to load some data

    ```
    <copy>
    INSERT INTO oratt.readtab VALUES (1,'Hello');
    INSERT INTO oratt.readtab VALUES (2,'World');
    INSERT INTO oratt.writetab VALUES (100, 'TimesTen');
    INSERT INTO oratt.writetab VALUES (101, 'Cache');
    COMMIT;
    </copy>
    ```
4. Use SQL*Plus session to grant the SELECT privilege on the `oratt.readtab` table
and the SELECT, INSERT, UPDATE, and DELETE privileges on the `oratt.writetab` table
to the cache administration user (cacheuser2, in this example).

    ```
    <copy>
    GRANT SELECT ON oratt.readtab TO cacheuser2;
    GRANT SELECT ON oratt.writetab TO cacheuser2;
    GRANT INSERT ON oratt.writetab TO cacheuser2;
    GRANT UPDATE ON oratt.writetab TO cacheuser2;
    GRANT DELETE ON oratt.writetab TO cacheuser2;
    </copy>
    ```


You may now **proceed to the next lab**.

## Acknowledgements
* **Author** - Dario VEGA, February 2024
* **Last Updated By/Date** - Dario VEGA, February 2024
