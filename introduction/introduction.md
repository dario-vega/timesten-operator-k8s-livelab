# Introduction

## About this Workshop

*Oracle TimesTen In-Memory Database* (TimesTen) is a lightweight, fully persistent, and highly available in-memory
relational database that delivers microsecond response and high throughput for OLTP applications.
You can use TimesTen as a database of record or as a cache for Oracle Database.

TimesTen supports SQL, standard APIs, complete ACID properties, and highly available replication mechanisms.

**TimesTen Offerings**

TimesTen has the following offerings:

* *TimesTen Classic* is a memory-optimized relational database that provides responsiveness and high throughput for applications. Through transactional replication, TimesTen Classic offers high availability for the in-memory database.
* *TimesTen Cache* is ideal for caching performance-critical subsets of an Oracle Database for improved response time in the application tier. Cache tables can be read-only or updatable. Applications automatically read and update the cache tables using standard SQL and data synchronization between the cache and the Oracle Database. Cache offers applications the full generality and functionality of a relational database, the transparent maintenance of cache consistency with the Oracle Database, and the high performance of an in-memory database.
* *TimesTen In-Memory Database for Kubernetes* offering on Oracle Cloud Marketplace enables TimesTen deployment on Oracle Cloud Infrastructure (OCI) Kubernetes Engine (OKE) or on on-premises infrastructure. When TimesTen is combined with *TimesTen Kubernetes Operator (TimesTen Operator)*, TimesTen provides a simple and robust solution for high performance, highly available data management in Kubernetes environments. It can quickly deploy TimesTen in standalone mode or as a cache (for Oracle Database).

**TimesTen Performance Overview**

TimesTen delivers high performance by changing the assumptions about where data resides at runtime.
The TimesTen in-memory database manages data in memory, optimizes data structures, and accesses algorithms accordingly.
Thus, database operations run with maximum efficiency, achieving dramatic gains in responsiveness and throughput,
even compared with a fully cached, file system-based relational database management system (RDBMS).


* Much of the work that is done by a conventional RDBMS is done under the assumption that data primarily resides in the file system.
Optimization algorithms, buffer pool management, and indexed retrieval techniques are designed based on this fundamental assumption.

* TimesTen is designed with the knowledge that data resides in main memory and can take more direct routes to data, reducing the length
of the code path and simplifying algorithms and structure.

* When the assumption of the file system is removed, complexity is dramatically reduced. The number of machine instructions drops, buffer pool management disappears, extra data copies are not needed, index pages shrink, and their structure is simplified. The design becomes simple and more compact, and requests are processed faster. The figure shows the simplicity of the TimesTen design.

    ![Comparing a File System-Based RDBMS to TimesTen Classic](images/in-memory-architecture.png)

**Deployment of Active Standby Pairs with the TimesTen Operator**

The *TimesTen Operator* enables simple deployment of your active standby pairs of TimesTen Classic databases and automated failure detection and recovery. You can:

* Decide you want to deploy a new replicated pair of TimesTen databases.
* Decide the attributes of those databases.
* Create the configuration files for those attributes.
* Use the `kubectl create` command to create a TimesTenClassic object to represent the replicated pair.
* Use the `kubectl get` and `kubectl describe` commands to observe the provisioning of the active standby pair.

Applications that run in other pods use TimesTen's standard client/server drivers to access TimesTen databases.

You **do not have** to monitor the TimesTen databases continually, configure replication, perform failover, or re-duplicate a database after failure.
**The TimesTen Operator performs all these functions and works to keep the databases up and running with minimal effort on your part.**

This workshop shows you how to deploy and run TimesTen inside a Kubernetes cluster using the *TimesTen Operator*. In this lab, we will set up *OKE* and perform the following:  

* Deploy an active standby pair.
* Connect TimesTen Cache to Oracle Containerized Database.
* Connect TimesTen Cache to Oracle Autonomous Database.
* Connect TimesTen Cache to Oracle Database Cloud Service.

### Objectives

* Set up a Container Engine for Kubernetes on the Oracle Cloud Infrastructure
* Install the TimesTen Kubernetes Operator
* Configure and launch a Timesten Database on Kubernetes
* Configure and launch a Oracle Autonomous Database
* Configure and launch a Timesten Cache Database on Kubernetes
* Destroy the environments

### Prerequisites

* Access to an Oracle Cloud account

You may now **proceed to the next lab**.

## Learn More

* [Oracle TimesTen In-Memory Database Kubernetes Operator User's Guide](https://docs.oracle.com/en/database/other-databases/timesten/22.1/kubernetes-operator/index.html)

## Acknowledgements
* **Author** - Dario VEGA, February 2023
* **Last Updated By/Date** - Dario VEGA, February 2023
