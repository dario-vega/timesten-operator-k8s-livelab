# Introduction

## About this Workshop

*Oracle TimesTen In-Memory Database* is a lightweight, fully featured, relational in-memory database
that offers **unrivalled performance**, simple installation and management, and high performance high-availability.
It is well suited to running in containerized environments and, combined with the *TimesTen Kubernetes Operator*,
provides a simple and robust solution for high performance, highly available data management in Kubernetes environments.

The *TimesTen Operator* is designed for simple deployment of your active standby pairs of *TimesTen Classic databases*
and for automated failure detection and recovery. For example,
* You decide you want to deploy a new replicated pair of TimesTen databases.
* You decide the attributes of those databases.
* You create the configuration files for those attributes.
* You use the `kubectl create` command to create a TimesTenClassic object to represent the replicated pair.
* You use the `kubectl get` and `kubectl describe` commands to observe the provisioning of the active standby pair.
* Applications that run in other Pods use TimesTen's standard client/server drivers to access TimesTen databases.

You **do not have** to monitor the TimesTen databases continually, configure replication, perform failover,
or re-duplicate a database after failure. **The TimesTen Operator performs all these functions and works
to keep the databases up and running with minimal effort on your part**.

This lab shows you how to deploy and run an *Oracle TimesTen* inside a Kubernetes
cluster, using the *Oracle TimesTen In-Memory Database Kubernetes Operato*r.  
We will setup a *Container Engine for Kubernetes (OKE)* and deploy
- Active Standby Pair
- TimesTen Cache connected to Oracle Containerized Database.
- TimesTen Cache connected to Oracle Autonomous Database.
- TimesTen Cache connected to Oracle Database Cloud Service.

### Objectives

* Set up a Container Engine for Kubernetes on the Oracle Cloud Infrastructure
* Install the TimesTen Kubernetes Operator
* Configure and launch a Timesten Database on Kubernetes
* Configure and launch a Oracle Autonomous Database on Kubernetes
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
