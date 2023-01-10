---
title: Failover an application
weight: 500
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration, failover an application
description: How to failover an application from one Kubernetes cluster to another.
series: setup-AysncDR
---


In case of a disaster, where one of your Kubernetes clusters is down and inaccessible, you can failover the applications running on it to an operational Kubernetes cluster. To achieve this, you should stop your application on the source cluster and start the application on an active Kubernetes cluster. 

The following considerations are used in the examples on this page. Update them to the appropriate values for your environment.
 
* **Source Cluster** is the Kubernetes cluster which is down and where your applications were originally running.
* **Destination Cluster** is the Kubernetes cluster where the applications will be failed over.
* The Zookeeper application is being failed over to the destination cluster.



## Stop the application on the source cluster (if accessible or applicable)

If your source Kubernetes cluster is still alive and is accessible, {{<companyName>}} recommends you to stop the applications before failing them over to the destination cluster.

{{<info>}}**NOTE:** Skip this section if you are using Stork version 2.9.0 or newer. As you would have already enabled the `autoSuspend` feature as explained in the [previous section](/operations/operate-kubernetes/disaster-recovery/async-dr/schedule-migration/#schedule-a-migration-from-your-source-cluster). This feature will automatically suspend your migration schedules on the source cluster. Therefore, proceed to the [next section](#start-the-application-on-the-destination-cluster).{{</info>}}


If you are using an older version of Stork, you need to stop the applications by manually changing the replica count of your deployments and statefulsets to 0. In this way, your application resources will persist in Kubernetes, but the actual application will not be running. 

1. Scale down the replica count of your application:

  ```text
  kubectl scale --replicas 0 statefulset/zk -n zookeeper
  ```
    As the `zookeeper` namespace is being used in the above command, it will scale down the replica count for the Zookeeper application. Update the namespace to your application namespace.


2. Run the following command to suspend the migration schedule. Once the replicas for your application's statefulset are set to 0, you need to suspend the migration schedule on the source cluster. This is done so that your application's stateful sets are not updated to 0 replicas on the destination cluster:

  ```text
  storkctl suspend migrationschedule migrationschedule -n <migrationnamespace>
  ```

3. Verify if the schedule has been suspended:

  ```text
  storkctl get migrationschedule -n <migrationnamespace>
  ```

  ```output
  NAME                POLICYNAME   CLUSTERPAIR     SUSPEND   LAST-SUCCESS-TIME     LAST-SUCCESS-DURATION
  migrationschedule   <your-schedule-policy>   remotecluster   true      01 Dec 22 23:31 UTC   10s
  ```


## Start the application on the destination cluster

You can allow Stork to activate migration either on all namespaces or one namespace at a time. For performance reasons, if you have a high number of namespaces in your migration schedule, {{<companyName>}} recommends you migrate one namespace at a time.  

1. Each application spec will have the annotation `stork.openstorage.org/migrationReplicas` indicating the replica count on the source cluster. Run the following command to update the replica count of your app to the same number as on your source cluster:

  ```text
  storkctl activate migration -n zookeeper
  ```
    Run the following command to migrate all namespaces:

    ```text
    storkctl activate migration --all-namespaces
    ```

    Stork will look for that annotation and scale it to the correct number automatically. Once the replica count is updated, the application will start running, and the failover will be completed.



2. Verify that your application is up and running:

  ```text
  kubectl get pods -n zookeeper
  ```

  ```output
  NAME   READY   STATUS    RESTARTS      AGE
  zk-0   1/1     Running   0             3m18s
  zk-1   1/1     Running   0             2m54s
  zk-2   1/1     Running   0             99s
  ```
    You can see that the status of all the pods for Zookeeper shows running, indicating that your application is operational.
