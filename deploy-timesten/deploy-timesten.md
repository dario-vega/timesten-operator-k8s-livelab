# Deploy an Active Standby Pair Example

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
    <copy>mkdir -p cm_mytimestendb</copy>
    ```
2. Navigate to the ConfigMap directory.

    ```
    <copy>cd cm_mytimestendb</copy>
    ```
3. Create the db.ini file in this ConfigMap directory (cm_sample, in this example). In this db.ini file, define the PermSize and DatabaseCharacterSet connection attributes.

    ```
    <copy>vi db.ini</copy>
    ```
    ```
    <copy>
    PermSize=200
    DatabaseCharacterSet=AL32UTF8
    </copy>
    ```

4. Create the schema.sql file in this ConfigMap directory.
    ```
    <copy>vi schema.sql</copy>
    ```
    ```
    <copy>
    create sequence sampleuser.s;
    create table sampleuser.emp (
      id number not null primary key,
      name char(32)
    );
    </copy>
    ```

5. Create the ConfigMap.

    ```  
    <copy>kubectl create configmap mytimestendbconf --from-file=cm_mytimestendb</copy>

    configmap/mytimestendbconf created
    ```
    You successfully created and deployed the mytimestendbconf ConfigMap.

6. Use the kubectl describe command to verify the contents of the ConfigMap.
    ```
    <copy>kubectl describe configmap mytimestendbconf</copy>

    Name:         mytimestendbconf
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Data
    ====

    db.ini:
    ----
    PermSize=200
    DatabaseCharacterSet=AL32UTF8

    schema.sql:
    ----
    create sequence appuser.s;
    create table appuser.emp (
        id number not null primary key,
        name  char(32)
    );

    BinaryData
    ====

    Events:  <none>

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
    <copy>vi mytimestendb.yaml</copy>
    ```

    ```
    <copy>
    apiVersion: timesten.oracle.com/v1
    kind: TimesTenClassic
    metadata:
        name: mytimestendb
    spec:
        ttspec:
    storageClassName: oci-bv
    storageSize: 50Gi
    image: iad.ocir.io/oradbclouducm/demos-tt/timesten/timesten:22.1.1.18.0
    imagePullSecret: ocirsecret
    cacheCleanup: false
    dbConfigMap:
    - mytimestendbconf
    dbSecret:
    - mysecret
    </copy>
    ```
2. Use the kubectl create command to create the TimesTenClassic object from the contents of the YAML file.
Doing so begins the process of deploying your active standby pair of TimesTen databases in the Kubernetes cluster.

    ```
    <copy>kubectl create -f mytimestendb.yaml</copy>
    timestenclassic.timesten.oracle.com/mytimestendb created
    ```


You may now **proceed to the next lab**.

## Acknowledgements
* **Author** - Dario VEGA, February 2024
* **Last Updated By/Date** - Dario VEGA, February 2024
