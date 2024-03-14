# Deploy TimesTen Cache in TimesTen Classic Example

## Introduction


Estimated Time: 20 minutes

### Objective
* Create a database running on Kubernetes, using a block volume as persistency store

### Prerequisites
* You have executed Lab 2: Install the Oracle Database Kubernetes Operator

## Task 1: Create a ConfigMap Object

This section creates the sample ConfigMap. This ConfigMap contains the db.ini, the adminUser, and the schema.sql metadata files.
This ConfigMap will be referenced when you define the TimesTenClassic object.

On your Linux development host:

1. From the directory of your choice, create an empty subdirectory for the metadata files.
This example creates the cm_mycachedb subdirectory.

    ```
    <copy>
    mkdir -p cm_mycachedb
    cd cm_mycachedb
    </copy>
    ```
2. Navigate to the ConfigMap directory.

    ```
    <copy>cd cm_mycachedb</copy>
    ```
3. Create the db.ini file in this ConfigMap directory (cm_sample, in this example). In this db.ini file, define the PermSize and DatabaseCharacterSet connection attributes.

    ```
    <copy>vi db.ini</copy>
    ```
    ```
    <copy>
    PermSize=200
    DatabaseCharacterSet=AL32UTF8
    OracleNetServiceName=cache_high
    </copy>
    ```

4. Create the cacheuser file in this ConfigMap directory.

    ```
    <copy>vi cacheUser</copy>
    ```
    ```
    <copy>
    cacheuser2/ttpwd/Oraclepwd##2020
    </copy>
    ```
5. Create the schema.sql file in this ConfigMap directory.
    ```
    <copy>vi schema.sql</copy>
    ```
    ```
    <copy>
    create sequence appuser.s;
    create table appuser.emp (
      id number not null primary key,
      name char(32)
    );
    create user oratt identified by ttpwd;
    grant admin to oratt;
    </copy>
    ```

6. Create the schema.sql file in this ConfigMap directory.
    ```
    <copy>vi cachegroups.sql</copy>
    ```
    ```
    <copy>
    CREATE DYNAMIC ASYNCHRONOUS WRITETHROUGH CACHE GROUP writecache
    FROM oratt.writetab (pk NUMBER NOT NULL PRIMARY KEY,attr VARCHAR2(40));

    CREATE READONLY CACHE GROUP readcache
    AUTOREFRESH
      INTERVAL 5 SECONDS
    FROM oratt.readtab (keyval NUMBER NOT NULL PRIMARY KEY,str VARCHAR2(32));

    LOAD CACHE GROUP readcache COMMIT EVERY 256 ROWS;
    </copy>
    ```

7. Create the ConfigMap.

    ```  
    <copy>kubectl create configmap mycacheconf --from-file=cm_mycachedb</copy>

    configmap/mycacheconf created
    ```
    You successfully created and deployed the mytimestendbconf ConfigMap.

8. Use the kubectl describe command to verify the contents of the ConfigMap.
    ```
    <copy>kubectl describe configmap mycacheconf</copy>

    Name:         mycacheconf
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Data
    ====
    schema.sql:
    ----
    create sequence sampleuser.s;
    create table sampleuser.emp (
      id number not null primary key,
      name char(32)
    );
    create user oratt identified by ttpwd;
    grant admin to oratt;

    cacheCleanup:
    ----
    false

    cacheUser:
    ----
    cacheuser2/ttpwd/Oraclepwd##2020

    cachegroups.sql:
    ----
    CREATE DYNAMIC ASYNCHRONOUS WRITETHROUGH CACHE GROUP writecache
    FROM oratt.writetab (pk NUMBER NOT NULL PRIMARY KEY,attr VARCHAR2(40));

    CREATE READONLY CACHE GROUP readcache
    AUTOREFRESH
      INTERVAL 5 SECONDS
    FROM oratt.readtab (keyval NUMBER NOT NULL PRIMARY KEY,str VARCHAR2(32));

    LOAD CACHE GROUP readcache COMMIT EVERY 256 ROWS;

    db.ini:
    ----
    PermSize=200
    DatabaseCharacterSet=AL32UTF8
    OracleNetServiceName=cache_high


    ```

## Task 2: Create a Secret Object

This section creates the sample Secret. This Secret contains the adminUser.
This Secret will be referenced when you define the TimesTenClassic object.

On your Linux development host:

1. From the directory of your choice, create an empty subdirectory for the metadata files. This example creates the secret_mysecret subdirectory.

    ```
    <copy>mkdir -p secret_mysecret</copy>
    ```
2. Navigate to the secret_mysecret directory.

    ```
    <copy>cd secret_mysecret</copy>
    ```
3. Create the adminUser file in this directory.

    ```
    <copy>vi adminUser</copy>
    ```
    ```
    <copy>sampleuser/samplepw</copy>
    ```

4. Create the Secret.

    ```  
    <copy>kubectl create secret generic mysecret --from-file=secret_mysecret</copy>
    secret/mysecret created
    ```
    You successfully created and deployed the mytimestendbconf ConfigMap.

5. Use the kubectl describe command to verify the contents of the Secret.
    ```
    <copy>kubectl describe secret mysecret</copy>

    Name:         mysecret
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Type:  Opaque

    Data
    ====
    adminUser:  17 bytes

    ```


## Task 3: Create a TimesTenClassic Object

This section creates the TimesTenClassic object.

On your Linux development host:

1. Create an empty YAML file. The YAML file contains the definitions for the TimesTenClassic object.

    ```
    <copy>vi cachedb.yaml</copy>
    ```

    ```
    <copy>
    apiVersion: timesten.oracle.com/v1
    kind: TimesTenClassic
    metadata:
      name: cachedb
    spec:
      ttspec:
        storageClassName: oci-bv
        storageSize: 50Gi
        image: container-registry.oracle.com/timesten/timesten:22.1.1.19.0
        imagePullSecret: sekret
        cacheCleanup: false
        dbConfigMap:
        - mycacheconf
        - tns
        dbSecret:
        - mysecret
    </copy>
    ```
2. Use the kubectl create command to create the TimesTenClassic object from the contents of the YAML file.
Doing so begins the process of deploying your active standby pair of TimesTen databases in the Kubernetes cluster.

    ```
    <copy>kubectl create -f cachedb.yaml</copy>
    timestenclassic.timesten.oracle.com/cachedb created
    ```



## Task 4: Monitor the State of a TimesTenClassic Object

Use the `kubectl get` and the `kubectl describe` commands to monitor the progress of the active standby pair as it is provisioned.

1. Use the `kubectl get` command and review the STATE field. Observe the value is Initializing. The active standby pair provisioning has begun, but is not yet complete.

    ```
    <copy>kubectl get timestenclassic cachedb</copy>
    ```

2. Use the `kubectl describe` command to view the initial provisioning in detail.

    ```
    <copy>kubectl describe timestenclassic cachedb</copy>
    ```

3. Use the kubectl get command again to see if value of the STATE field has changed.
In this example, the value is Normal, indicating the active standby pair of databases are now provisioned and the process is complete.

    ```
    <copy>kubectl get ttc cachedb</copy>

    NAME      STATE          ACTIVE   AGE
    cachedb   Initializing   None     7m34s
    ```

4. Use the kubectl describe command again to view the active standby pair provisioning in detail.

    ```
    <copy>kubectl describe timestenclassic cachedb</copy>
    ```

    Your active standby pair of *TimesTen* databases are successfully deployed (as indicated by `Normal`).

    There are two TimesTen databases, configured as an active standby pair.
    * One database is active. In this example, `cachedb-0` is the active database, as indicated by Rep State `ACTIVE`.
    * The other database is standby. In this example, `cachedb-1` is the standby database as indicated by Rep State `STANDBY`.
    The active database can be modified and queried. Changes made on the active database are replicated to the standby database.

    If the active database fails, the *Operator* automatically promotes the standby database to be the active.
    The formerly active database will be repaired or replaced, and will then become the standby.


## Task 5: Verify Connection to Active Database

You can run the `kubectl exec` command to invoke shells in your Pods and control *TimesTen*, which is running in those Pods.
By default, TimesTen runs in the Pods as the `timesten` user. Once you have established a shell in the Pod, verify you can
connect to the sample database, and that the information from the metadata files is correct.
You can optionally run queries against the database or any other operations.

1. Establish a shell in the Pod.

    ```
    <copy>kubectl exec -it cachedb-0 -c tt -- /bin/bash</copy>
    ```

2. Connect to the `mytimestendb` database. Attempt to connect to the database as the `sampleuser` user.
Check that the `PermSize` value of 200 is correct. Check that the `sampleuser.emp` table exists.

    ```
    <copy>ttIsql cachedb</copy>
    ```

    ```
    <copy>cachegroups;</copy>

    Cache Group CACHEUSER2.READCACHE:

      Cache Group Type: Read Only
      Autorefresh: Yes
      Autorefresh Mode: Incremental
      Autorefresh State: On
      Autorefresh Interval: 5 Seconds
      Autorefresh Status: ok
      Aging: No aging defined

      Root Table: ORATT.READTAB
      Table Type: Read Only

    Cache Group CACHEUSER2.WRITECACHE:

      Cache Group Type: Asynchronous Writethrough (Dynamic)
      Autorefresh: No
      Aging: LRU on

      Root Table: ORATT.WRITETAB
      Table Type: Propagate

    2 cache groups found.
    ```

    ```
    <copy>SELECT * FROM oratt.readtab;</copy>

    < 1, Hello >
    < 2, World >
    ```

    3. exit from the `ttIsql` command

## Task 6: Perform Operations On Cache Group Tables

The examples in this section perform operations on the oratt.readtab and the oratt.writetab tables to verify that TimesTen Cache is working properly.
* Perform Operations on the oratt.readtab Table
* Perform Operations on the oratt.writetab Table

Let start with **Perform Operations on the oratt.readtab Table**

1. connect to your Oracle Database using SQL*Plus as the schema user (oratt, in this example).
Then, insert a new row, delete an existing row, and update an existing row in the oratt.readtab table of the Oracle Database and commit the changes.

    ```
    <copy>
    sqlplus oratt/Oraclepwd##2020@cache_high;
    </copy>
    ```

    ```
    <copy>
    INSERT INTO oratt.readtab VALUES (3,'Welcome');
    DELETE FROM oratt.readtab WHERE keyval=2;
    UPDATE oratt.readtab SET str='Hi' WHERE keyval=1;
    COMMIT;
    SELECT * FROM oratt.readtab;
    exit
    </copy>
    ```

    Since the read-only cache group was created with an autorefresh interval of 5 seconds,
    the TimesTen oratt.readtab cache table in the readcache cache group is automatically
    refreshed after 5 seconds with the committed updates from the cached oratt.readtab table of the Oracle Database.
    The next step is to test that the data was correctly propagated from the Oracle Database to the TimesTen database.

2. Use the TimesTen ttIsql utility to connect to the cachetest database.
Query the TimesTen oratt.readtab table to verify that the table has been updated with the committed updates from the Oracle Database.


    ```
    <copy>ttIsql cachedb</copy>
    ```
    ```
    <copy>SELECT * FROM oratt.readtab;</copy>

    < 1, Hello >
    < 2, World >
    ```

Now with **Perform Operations on the oratt.writetab Table**

3. Use the TimesTen ttIsql utility to connect to the cachetest database.
Query the TimesTen oratt.writetab table. Recall that the writecache cache group is a dynamic cache group.
Thus by issuing the SELECT statement, the cache instance is automatically loaded from the cached Oracle Database table,
if the data is not found in the TimeTen cache table.


    ```
    <copy>ttIsql "DSN=cachedb;UID=cacheuser2;PWD=ttpwd;OraclePWD=Oraclepwd##2020";</copy>
    ```
    ```
    <copy>
    SELECT * FROM oratt.writetab WHERE pk=100;
    INSERT INTO oratt.writetab VALUES (102,'Cache');
    DELETE FROM oratt.writetab WHERE pk=101;
    UPDATE oratt.writetab SET attr='Oracle' WHERE pk=100;
    select * from oratt.writetab;
    </copy>
    ```
    ```
    <copy>
    sqlplus oratt/Oraclepwd##2020@cache_high;
    </copy>
    ```

    ```
    <copy>
    SELECT * FROM oratt.writetab ORDER BY pk;
    exit
    </copy>
    ```

You may now **proceed to the next lab**.

## Acknowledgements
* **Author** - Dario VEGA, February 2024
* **Last Updated By/Date** - Dario VEGA, February 2024
