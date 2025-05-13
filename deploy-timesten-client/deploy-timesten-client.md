# Deploy TimesTen Client

## Introduction


Estimated Time: 20 minutes

### Objective
* Create a client Pod running on Kubernetes, this Pod contains the TimesTen client installed.

### Prerequisites
* You have executed Lab 2: Install the Oracle Database Kubernetes Operator


## Task 1: Create a ConfigMap Object for the Database Network configuration

This section creates the sample ConfigMap. This ConfigMap contains the tnsnames configuration if using a Oracle DB.

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
3. Create the sqlnet.ora file in this ConfigMap directory.
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

4. Create the ConfigMap.

    ```  
    <copy>
    cd ..
    kubectl create configmap tns --from-file=cm_tns
    </copy>

    configmap/tns created
    ```
    You successfully created and deployed the `tns` ConfigMap.

5. Use the kubectl describe command to verify the contents of the ConfigMap.
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

    BinaryData
    ====

    Events:  <none>

    ```

## Task 2: Create a Client Pod with the timesten client deployed

This section creates the TimesTen pod object with the timesten client deployed.

On your Linux development host:

1. Create an empty YAML file. The YAML file contains the definitions for the TimesTenClassic object.

    ```
    <copy>vi client.yaml</copy>
    ```

    ```
    <copy>
    apiVersion: v1
    kind: Pod
    metadata:
      name: client
      labels:
        name: client
    spec:
      imagePullSecrets:
      - name: sekret
      initContainers:
      - name: init
        image: container-registry.oracle.com/timesten/timesten:latest
        imagePullPolicy: Always
        volumeMounts:
        - name: tt
          mountPath: /tt
        command:
        - sh
        - "-c"
        - |
          /bin/bash <<'EOF'
          exit 0
          EOF
      containers:
      - name: client
        image: container-registry.oracle.com/timesten/timesten:latest
        imagePullPolicy: Always
        volumeMounts:
        - name: tt
          mountPath: /tt
        - name: tns-conf
          mountPath: /tns          
        command:
        - sh
        - "-c"
        - |
          /bin/bash <<'EOF'
          while [ 1 ]
          do
              sleep 15
          done
          EOF
      volumes:
      - name: tt
        emptyDir: {}
      - name: tns-conf
        configMap:
          name: tns
    </copy>
    ```
2. Use the kubectl create command to create the TimesTenClassic object from the contents of the YAML file.

    ```
    <copy>kubectl create -f client.yaml</copy>
    pod/client created created
    ```
3. Establish a shell in the Pod.

    ```
    <copy>kubectl exec -it client -c client -- /bin/bash</copy>
    ```

4. Using a text editor, edit the `sys.odbc.ini` file so that the file from [ODBC Data Sources] onwards looks as follows
    ```
    <copy>vi $TIMESTEN_HOME/conf/sys.odbc.ini</copy>
    ```
    ```
    <copy>
    [mytimestendbCS]
    TTC_SERVER_DSN=mytimestendb
    TTC_SERVER=mytimestendb-0.mytimestendb.default.svc.cluster.local/6625
    TTC_SERVER2=mytimestendb-1.mytimestendb.default.svc.cluster.local/6625
    ConnectionCharacterSet=AL32UTF8
    </copy>
    ```

5. Connect to the TimesTen database.

    ```
    <copy>ttIsqlCS -connStr "DSN=mytimestendbCS;uid=sampleuser;pwd=samplepw"</copy>

    Copyright (c) 1996, 2023, Oracle and/or its affiliates. All rights reserved.
    Type ? or "help" for help, type "exit" to quit ttIsql.


    Enter password for 'sampleuser':
    Connection successful: DSN=mytimestendbCS;TTC_SERVER=mytimestendb-0.mytimestendb.default.svc.cluster.local;TTC_SERVER_DSN=mytimestendb;UID=appuser;DATASTORE=/tt/home/timesten/datastore/mytimestendb;DATABASECHARACTERSET=AL32UTF8;CONNECTIONCHARACTERSET=AL32UTF8;AUTOCREATE=0;PERMSIZE=200;DDLREPLICATIONLEVEL=3;FORCEDISCONNECTENABLED=1;
    (Default setting AutoCommit=1)

    ```

    ```
    <copy>tables</copy>

    SAMPLEUSER.EMP
    1 table found.
    sampleuser: Command>
    ```

    ```
    <copy>
    SELECT * FROM emp;
    </copy>
    ```

6. exit from the `ttIsqlCS` command and from the `shell` command

7. Delete the client pod.

    ```
    <copy>kubectl delete -f client.yaml</copy>
    timestenclassic.timesten.oracle.com/cachedb created
    ```

8. Delete the ConfigMap.

    ```  
    <copy>
    kubectl delete configmap tns
    </copy>

    configmap "tns" deleted
    ```

9. Delete the files created.

    ```
    <copy>
    rm client.yaml
    rm -rf cm_tns
    </copy>
    ```

You may now **proceed to the next lab**.

## Acknowledgements
* **Author** - Dario VEGA, February 2024
* **Last Updated By/Date** - Dario VEGA, May 2025
