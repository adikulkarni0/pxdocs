---
title: Schedule a migration
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration
description: How to set up asynchronous schedules.
series: setup-AysncDR
weight: 400
---

Once your source and destination clusters are paired, you need to create a schedule to migrate Kubernetes resources between them. This can be achieved by setting a schedule policy that migrates Kubernetes resources at regular intervals. This interval should be set depending on your network speed and the amount of data to be transferred between clusters.


## Create a schedule policy on your source cluster

You can create either a `SchedulePolicy` or `NamespacedSchedulePolicy`. The `SchedulePolicy` is cluster-scoped (cluster wide) while the `NamespacedSchedulePolicy` is namespace-scoped.

Perform the following steps from your source cluster to create a schedule policy: 


{{<info>}}**NOTE:**

* {{<companyName>}} recommends using an interval of at least 15 minutes.
* If the specified interval time is not sufficient, Portworx will not start the next interval until the first interval is completed.
{{</info>}}


1. Create `SchedulePolicy` spec file:

    ```text
    apiVersion: stork.libopenstorage.org/v1alpha1
    kind: SchedulePolicy
    metadata:
      name: <your-schedule-policy>
    policy:
      interval:
        intervalMinutes: 30
    ```

    For a list of parameters that you can use to create a schedule policy, see the [Schedule Policy](/reference/crd/schedule-policy/) page.

2. Apply your policy:

    ```text
    kubectl apply -f <your-schedule-policy>.yaml
    ```

3. Verify if the policy has been created:

    ```text
    storkctl get schedulepolicy <your-schedule-policy>
    ```

    ```output
    NAME           INTERVAL-MINUTES   DAILY     WEEKLY             MONTHLY
    <your-schedule-policy>     30                 N/A       N/A                N/A
    ```
{{<info>}}**NOTE:** You can also use the prepackaged schedule policies that are available with your Portworx installation. Run the `storkctl get schedulepolicy` command to get the list of these policies, then specify a policy name in the next section for creating a migration schedule.{{</info>}}

## Schedule a migration from your source cluster

Once a policy has been created, you can use it to schedule a migration.

Perform the following steps from your source cluster:


1. Copy and paste the following spec into a file called `migrationschedule.yaml`. Modify the following spec to use a different migration schedule name and/or namespace. Ensure that the `clusterPair` name is correct:

  ```text
  apiVersion: stork.libopenstorage.org/v1alpha1
  kind: MigrationSchedule
  metadata:
    name: migrationschedule
    namespace: <migrationnamespace>
    annotations:
      # Add the below annotations when PX-Security is enabled on both the clusters
      #openstorage.io/auth-secret-namespace: <the namespace where the kubernetes secret holding the auth token resides>
      #openstorage.io/auth-secret-name: <the name of the kubernetes secret which holds the auth token>
  spec:
    template:
      spec:
        clusterPair: remotecluster
        includeResources: true
        startApplications: false
        includeVolumes: false
        namespaces:
        - zookeeper
    schedulePolicyName: <your-schedule-policy>
    suspend: false
    autoSuspend: true
  ```
  

    {{<info>}} **NOTE:**

* The option `startApplications` must be set to false in the spec. Otherwise, the first migration will start the pods on the remote cluster, and all subsequent migrations will fail because the volumes will already be in use.

* If you are running Stork 2.9.0 version or later, you can set the `autoSuspend` to `true`, as shown in the above spec. In case of a disaster, this will suspend the DR migration schedules automatically on your source cluster, and you will be able to migrate your application to an active Kubernetes cluster. If you are using an older version of Stork, refer to the [Failover an application](/operations/operate-kubernetes/disaster-recovery/async-dr/failover-app) page for achieving failover for your application.

* The auth annotations `openstorage.io/auth-secret-namespace` and `openstorage.io/auth-secret-name` must be set when PX-Security is enabled on both the source and destination cluster, as explained in the [ClusterPair](/operations/operate-kubernetes/disaster-recovery/async-dr/generate-apply-clusterpair/#generate-a-clusterpair-spec-on-the-destination-cluster) section.
    {{</info>}}


2. Apply your migration schedule:

  ```text
  kubectl apply -f migrationschedule.yaml
  ```
    If the policy name is missing or invalid, there will be events logged against the schedule object. Success and failures of the migrations created by the schedule will also result in events being logged against the object. These events can be seen by running a `kubectl describe` on the object

## Check your migration status from your source cluster

1. Run the following command from your source cluster to check the status of your migration. The following example uses Zookeeper as a demonstration application:

  ```text
  kubectl describe migrationschedules <migrationschedule-name> -n <migrationnamespace>
  ```

  ```output
  Name:         migrationschedule
  Namespace:    <migrationnamespace>
  Labels:       <none>
  Annotations:  <none>
  API Version:  stork.libopenstorage.org/v1alpha1
  Kind:         MigrationSchedule
  Metadata:
    Creation Timestamp:  2022-11-17T19:24:32Z
    Finalizers:
      stork.libopenstorage.org/finalizer-cleanup
    Generation:  43
    Managed Fields:
      API Version:  stork.libopenstorage.org/v1alpha1
      Fields Type:  FieldsV1
      fieldsV1:
        ........
        ............
      Manager:         stork
      Operation:       Update
      Time:            2022-11-17T19:24:33Z
    Resource Version:  23587269
    UID:               xxxxxxx-8d7763d99ed1
  Spec:
    Auto Suspend:          true
    Schedule Policy Name:  <your-schedule-policy>
    Suspend:               false
    Template:
      Spec:
        Admin Cluster Pair:                
        Cluster Pair:                      remotecluster
        Include Network Policy With CIDR:  <nil>
        Include Optional Resource Types:   <nil>
        Include Resources:                 true
        Include Volumes:                   <nil>
        Namespaces:
          zookeeper
        Post Exec Rule:           
        Pre Exec Rule:            
        Purge Deleted Resources:  <nil>
        Selectors:                <nil>
        Skip Deleted Namespaces:  <nil>
        Skip Service Update:      <nil>
        Start Applications:       false
  Status:
    Application Activated:  false
    Items:
      Interval:
        Creation Timestamp:  2022-11-17T23:55:11Z
        Finish Timestamp:    2022-11-17T23:56:39Z
        Name:                migrationschedule-interval-2022-11-17-235511
        Status:              Successful
        Creation Timestamp:  2022-11-18T00:25:20Z
        Finish Timestamp:    <nil>
        Name:                migrationschedule-interval-2022-11-18-002520
        Status:              Successful
  Events:
    Type     Reason      Age               From   Message
    ----     ------      ----              ----   -------
    Normal   Successful  59m               stork  Scheduled migration (migrationschedule-interval-2022-11-17-232511) completed successfully
    Normal   Successful  30m               stork  Scheduled migration (migrationschedule-interval-2022-11-17-235511) completed successfully
  ```
    * On a successful migration, you will see the same in the status of the migrated namespace. Also, other parameters, such as  `Auto Suspend` and `Schedule Policy Name` will be configured as previously.
    * The output of `kubectl describe` will also show the status of the migrations that were triggered for each of the policies along with the start and finish times. The statuses will be maintained for the last successful migration and any `Failed` or `InProgress` migrations for each policy type.

   
1. Run the following command to check if your migration is in progress. This command lists the names of migration objects with timestamp that are associated with your migration:

  ```text
  kubectl get migration -n <migrationnamespace>
  ```

  ```output
  NAME                                              AGE
  migrationschedule-interval-2022-11-18-002520      9m42s
  ```

1. Verify the status of your migration by running the following command on your destination cluster:


  ```text
  kubectl get statefulset -n <namespace>
  ```
  ```output
  NAME   READY   AGE
  zk     0/0     6m44s
  ```
    Log on to your remote cluster and confirm the respective namespace has been created, and respective applications are installed (the pods will not be running).
   
 


