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
    <copy>mkdir -p cm_mycachedb</copy>
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

4. Create the schema.sql file in this ConfigMap directory.
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

5. Create the schema.sql file in this ConfigMap directory.
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

6. Create the ConfigMap.

    ```  
    <copy>kubectl create configmap mycacheconf --from-file=cm_mycacheconf</copy>

    configmap/mycacheconf created
    ```
    You successfully created and deployed the mytimestendbconf ConfigMap.

6. Use the kubectl describe command to verify the contents of the ConfigMap.
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
    create sequence appuser.s;
    create table appuser.emp (
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
        image: iad.ocir.io/oradbclouducm/demos-tt/timesten/timesten:22.1.1.18.0
        imagePullSecret: ocirsecret
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


You may now **proceed to the next lab**.

## Acknowledgements
* **Author** - Dario VEGA, February 2024
* **Last Updated By/Date** - Dario VEGA, February 2024
