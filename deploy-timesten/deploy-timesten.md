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
    <copy>
    cd ..
    kubectl create configmap mytimestendbconf --from-file=cm_mytimestendb
    </copy>

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
    <copy>
    cd ..
    kubectl create secret generic mysecret --from-file=secret_mysecret
    </copy>
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
           image: container-registry.oracle.com/timesten/timesten:22.1.1.19.0
           imagePullSecret: sekret
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



## Task 4: Monitor the State of a TimesTenClassic Object

Use the `kubectl get` and the `kubectl describe` commands to monitor the progress of the active standby pair as it is provisioned.

1. Use the `kubectl get` command and review the STATE field. Observe the value is Initializing. The active standby pair provisioning has begun, but is not yet complete.

    ```
    <copy>kubectl get timestenclassic mytimestendb</copy>
    ```

2. Use the `kubectl describe` command to view the initial provisioning in detail.

    ```
    <copy>kubectl describe timestenclassic mytimestendb</copy>
    ```

3. Use the kubectl get command again to see if value of the STATE field has changed.
In this example, the value is Normal, indicating the active standby pair of databases are now provisioned and the process is complete.

    ```
    <copy>kubectl get ttc mytimestendb</copy>

    NAME           STATE    ACTIVE           AGE
    mytimestendb   Normal   mytimestendb-0   3m25s
    ```

4. Use the kubectl describe command again to view the active standby pair provisioning in detail.

    ```
    <copy>kubectl describe timestenclassic mytimestendb</copy>
    ```

    Your active standby pair of *TimesTen* databases are successfully deployed (as indicated by `Normal`).

    There are two TimesTen databases, configured as an active standby pair.
    * One database is active. In this example, `mytimestendb-0` is the active database, as indicated by Rep State `ACTIVE`.
    * The other database is standby. In this example, `mytimestendb-1` is the standby database as indicated by Rep State `STANDBY`.
    The active database can be modified and queried. Changes made on the active database are replicated to the standby database.

    If the active database fails, the *Operator* automatically promotes the standby database to be the active.
    The formerly active database will be repaired or replaced, and will then become the standby.

## Task 5: Verify Underlying Objects Exist

The Operator creates other underlying objects automatically. Verify that these objects are created.

1. Use the `kubectl get` command to verify the objects of type StatefulSet created

    ```
    <copy>kubectl get statefulset  mytimestendb</copy>
    ```

2. Use the `kubectl get` command to verify the objects of type Service created

    ```
    <copy>kubectl get service  mytimestendb</copy>
    ```
3. Use the `kubectl get` command to verify the objects of type pod created

    ```
    <copy>kubectl get pods -l app=mytimestendb</copy>
    ```

4. Use the `kubectl get` command to verify the objects of type PersistentVolumeClaims (PVCs) created

    ```
    <copy>kubectl get pvc -l app=mytimestendb</copy>
    ```
## Task 6: Verify Connection to Active Database

You can run the `kubectl exec` command to invoke shells in your Pods and control *TimesTen*, which is running in those Pods.
By default, TimesTen runs in the Pods as the `timesten` user. Once you have established a shell in the Pod, verify you can
connect to the sample database, and that the information from the metadata files is correct.
You can optionally run queries against the database or any other operations.

1. Establish a shell in the Pod.

    ```
    <copy>kubectl exec -it mytimestendb-0 -c tt -- /bin/bash</copy>
    ```

2. Connect to the `mytimestendb` database. Attempt to connect to the database as the `sampleuser` user.
Check that the `PermSize` value of 200 is correct. Check that the `sampleuser.emp` table exists.

    ```
    <copy>ttIsql mytimestendb</copy>

    connect "DSN=mytimestendb";
    Connection successful: DSN=mytimestendb;UID=timesten;DataStore=/tt/home/timesten/datastore/mytimestendb;DatabaseCharacterSet=AL32UTF8;ConnectionCharacterSet=US7ASCII;AutoCreate=0;PermSize=200;DDLReplicationLevel=3;ForceDisconnectEnabled=1;
    (Default setting AutoCommit=1)
    Command>
    ```

    ```
    <copy>connect adding "uid=sampleuser;pwd=samplepw" as sampleuser;</copy>

    Connection successful: DSN=mytimestendb;UID=sampleuser;DataStore=/tt/home/timesten/datastore/mytimestendb;DatabaseCharacterSet=AL32UTF8;ConnectionCharacterSet=US7ASCII;AutoCreate=0;PermSize=200;DDLReplicationLevel=3;ForceDisconnectEnabled=1;
    (Default setting AutoCommit=1)
    sampleuser: Command>
    ```

    ```
    <copy>tables</copy>

    SAMPLEUSER.EMP
    1 table found.
    sampleuser: Command>
    ```

    ```
    <copy>
    INSERT INTO sampleuser.emp VALUES (3,'Dario Vegas');
    UPDATE emp SET name='Dario VEGA' WHERE id=3;
    SELECT * FROM emp;
    </copy>
    ```

3. exit from the `ttIsql` command and from the `shell` command

4. execute the same commands using the pod `mytimestendb-1`

## Task 7: Handle Failover and Recovery in TimesTen Classic

The *Operator* automatically detects failures of the active TimesTen database and
the standby TimesTen database and works to fix any failures. When the Operator
detects a failure of the active database, it promotes the standby TimesTen database to be the active.
Client/server applications that are using the database are automatically reconnected to the new active database.
Transactions in flight are rolled back. Prepared statements need to be re-prepared by the applications.
The Operator will configure a new standby database.

This lab simulates a failure of the active TimesTen database.

1. Use the `kubectl delete pod` command to delete the active database (mytimestendb-0 in this case)

    ```
    <copy>kubectl delete pod mytimestendb-0</copy>
    ```

2. Use the `kubectl describe` command to observe how the Operator recovers from the failure.
The Operator promotes the standby database (`mytimestendb-1`) to be active. Any applications
that were connected to the `mytimestendb-0` database are automatically reconnected to the `mytimestendb-1` database by TimesTen.
After a brief outage, the applications can continue to use the database

    ```
    <copy>kubectl describe ttc mytimestendb</copy>
    ```
    ```
    <copy>kubectl get Events  --field-selector involvedObject.name=mytimestendb</copy>
    ```

    Kubernetes has automatically respawned a new mytimestendb-0 Pod to replace the Pod you deleted.
    The Operator configured TimesTen within that Pod, bringing the database in the Pod up as the new standby database.
    The replicated pair of databases are once again functioning normally.


You may now **proceed to the next lab**.

## Acknowledgements
* **Author** - Dario VEGA, February 2024
* **Last Updated By/Date** - Dario VEGA, February 2024
