# Deploy TimesTen Client

## Introduction


Estimated Time: 20 minutes

### Objective
* Create a database running on Kubernetes, using a block volume as persistency store

### Prerequisites
* You have executed Lab 2: Install the Oracle Database Kubernetes Operator


## Task 1: Create a TimesTenClassic Object

This section creates the TimesTenClassic object.

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
      - name: ocirsecret
      initContainers:
      - name: init
        image: iad.ocir.io/oradbclouducm/demos-tt/timesten/timesten:22.1.1.18.0
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
        image: iad.ocir.io/oradbclouducm/demos-tt/timesten/timesten:22.1.1.18.0
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
              sleep 60
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
Doing so begins the process of deploying your active standby pair of TimesTen databases in the Kubernetes cluster.

    ```
    <copy>kubectl create -f client.yaml</copy>
    timestenclassic.timesten.oracle.com/cachedb created
    ```
3. Using a text editor, edit the sys.odbc.ini file so that the file from [ODBC Data Sources] onwards looks as follows
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

4. Connect to the A/S pair database and verify that you have been automatically connected to the current active database.

    ```
    <copy>ttIsqlCS -connStr "DSN=mytimestendbCS;uid=appuser;pwd=samplepw"</copy>

    Copyright (c) 1996, 2023, Oracle and/or its affiliates. All rights reserved.
    Type ? or "help" for help, type "exit" to quit ttIsql.


    Enter password for 'appuser':
    Connection successful: DSN=mytimestendbCS;TTC_SERVER=mytimestendb-0.mytimestendb.default.svc.cluster.local;TTC_SERVER_DSN=mytimestendb;UID=appuser;DATASTORE=/tt/home/timesten/datastore/mytimestendb;DATABASECHARACTERSET=AL32UTF8;CONNECTIONCHARACTERSET=AL32UTF8;AUTOCREATE=0;PERMSIZE=200;DDLREPLICATIONLEVEL=3;FORCEDISCONNECTENABLED=1;
    (Default setting AutoCommit=1)
    Command>
    Command>
    Command> ^DDisconnecting...
    Done.
    ```

You may now **proceed to the next lab**.

## Acknowledgements
* **Author** - Dario VEGA, February 2024
* **Last Updated By/Date** - Dario VEGA, February 2024
