---
title: Asynchronous DR
linkTitle: Asynchronous DR
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration
description: How to achieve asynchronous DR across Kubernetes clusters using scheduled migrations
weight: 200
aliases:
    - /portworx-install-with-kubernetes/disaster-recovery/async-dr/
---
## Prerequisites

* **Version**: Portworx v2.1 or later needs to be installed on both clusters. Also requires Stork v2.2+ on both the clusters.
* **Secret Store** : Make sure you have configured a [secret store](/operations/key-management) on both your clusters. This will be used to store the credentials for the objectstore* **Network Connectivity**: Port 9001 on the destination cluster should be reachable by the source cluster.
* **Network Connectivity**: Port 9001 on the destination cluster should be reachable by the source cluster.
* **Stork helper**: `storkctl` is a command-line tool for interacting with a set of scheduler extensions.
{{< content "shared/portworx-install-with-kubernetes-disaster-recovery-stork-helper.md" >}}
* **Default Storage Class**: Make sure you have configured only one default storage class. Having multiple default storage classes will cause PVC migrations to fail.
* **License**: You will need a DR enabled Portworx license at both the source and destination cluster to use this feature.
* If the destination cluster runs on **GKE**, follow the steps in the [Migration with Stork on GKE](/operations/operate-kubernetes/migration/gke/) page.
* If the destination cluster runs on **EKS**, follow the steps in the [Migration with Stork on EKS](/operations/operate-kubernetes/migration/eks/) page.
* An external objectstore, such as Minio s3, AWS s3, GCE Object Storage, or Azure Blob Storage, must be setup outside the Kubernetes clusters. If an external objectstore is not provided, Portworx will create an internal objectstore with a 100G backing volume. **Pure Storage does not support using the internal objectstore in production.**

## Overview

With asynchronous DR, you can replicate Kubernetes applications and their data between two Kubernetes clusters. Here, a separate {{< pxEnterprise >}} cluster runs under each Kubernetes cluster.

* The active Kubernetes cluster asynchronously backs-up apps, configuration and data to a standby Kubernetes cluster.
* The standby Kubernetes cluster has running controllers, configuration and PVCs that map to a local volumes.
* Incremental changes in Kubernetes applications and Portworx data are continuously sent to the standby cluster.

The list of supported Kubernetes resources can be found [here](#supported-kubernetes-resources)
If a custom CR that you need is not present in this list, you can register a new one. Refer to the [Application Registration](/operations/operate-kubernetes/storage-operations/stateful-applications/application-registration/) document for instructions on how to do this.

## Enable load balancing on cloud clusters

If you're running Kubernetes on the cloud, you must configure an External LoadBalancer (ELB) for the Portworx API service.

{{<info>}}
**Warning:** Do not enable load balancing without authorization enabled on the Portworx cluster.
{{</info>}}

Enable load balancing by entering the `kubectl edit service` command and changing the service type value from `nodePort` to `loadBalancer`:

```text
kubectl edit service portworx-service -n kube-system
```
```output
kind: Service
apiVersion: v1
metadata:
  name: portworx-service
  namespace: kube-system
  labels:
    name: portworx
spec:
  selector:
    name: portworx
  type: loadBalancer
```

{{< content "shared/portworx-install-with-kubernetes-cluster-pair.md" >}}

## Schedule a migration

To schedule a migration, you must create a schedule policy or a namespaced schedule policy. The `NamespacedSchedulePolicy` is namespace-scoped while `SchedulePolicy` is cluster-scoped.

### Create a schedule policy

1. Paste the following content into a file called `testpolicy.yaml`:

    ```text
    apiVersion: stork.libopenstorage.org/v1alpha1
    kind: SchedulePolicy
    metadata:
      name: testpolicy
    policy:
      interval:
        intervalMinutes: 60
      daily:
        time: "10:14PM"
      weekly:
        day: "Thursday"
        time: "10:13PM"
      monthly:
        date: 14
        time: "8:05PM"
    ```

    For details about how you can configure a schedule policy, see the [Schedule Policy](/reference/crd/schedule-policy/) reference page.

2. Apply your spec by entering the following command:

    ```text
    kubectl apply -f testpolicy.yaml
    ```

3. Display your schedule policy. Enter the `storkctl get` command passing it the name of your policy:

    ```text
    storkctl get schedulepolicy
    ```

    ```output
    NAME           INTERVAL-MINUTES   DAILY     WEEKLY             MONTHLY
    testpolicy     60                 10:14PM   Thursday@10:13PM   14@8:05PM
    ```

### Create a namespaced schedule policy

1. Paste the following content into a file called `namespaced-testpolicy.yaml`:

    ```text
    apiVersion: stork.libopenstorage.org/v1alpha1
    kind: NamespacedSchedulePolicy
    metadata:
      name: testpolicy
      namespace: <namespace-for-sched-policy>
    policy:
      interval:
        intervalMinutes: 60
      daily:
        time: "10:14PM"
      weekly:
        day: "Thursday"
        time: "10:13PM"
      monthly:
        date: 14
        time: "8:05PM"
    ```

2. Apply your spec by entering the following command:

    ```text
    kubectl apply -f namespaced-testpolicy.yaml
    ```

3. Display your namespacedschedule policy. Enter the `kubectl -n <namespace-for-sched-policy>  get` command passing it the name of your policy:

    ```text
    kubectl -n <namespace-for-sched-policy>  get namespacedschedulepolicy
    ```

    ```output
    NAME           AGE
    testpolicy     1s
    ```

### Create a migration schedule

Once a schedule policy or namespaced schedule policy has been created, you can use it to schedule a migration. The spec for the MigrationSchedule spec contains the same fields as the Migration spec with the addition of the policy name. The MigrationSchedule object is namespaced like the Migration object.
Stork will first look for namespaced schedule policy in the same namespace of migration/backup. If a namespaced schedule policy with the specified name is not found it will look for global SchedulePolicy with the same name.

Note that `startApplications` should be set to false in the spec. Otherwise, the first Migration will start the pods on the remote cluster and will succeed. But all subsequent migrations will fail since the volumes will be in use.

{{<info>}}
**NOTE:** If your cluster has a DR license applied to it, you can only perform migrations in DR mode; this includes operations involving the `pxctl cluster migrate` command.
{{</info>}}

Continuing our previous example with `testpolicy`, here is how to create a `MigrationSchedule` object that schedules a migration:

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: MigrationSchedule
metadata:
  name: mysqlmigrationschedule
  namespace: mysql
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
      namespaces:
      - mysql
  schedulePolicyName: testpolicy
```

A few things to note: 
* The option `startApplications` is set to false so that the applications do not start when the resources are migrated. This is because the applications on the destination cluster should start only when the application fails over.
* The auth annotations `openstorage.io/auth-*` should be set when both source and destination clusters have PX-Security enabled. These annotations should point to the kubernetes secret which will hold the token required for authorizing migrations of Portworx volumes. Use a token which will authorize for all the volumes in namespaces this migration is handling. 

If the policy name is missing or invalid there will be events logged against the schedule object. Success and failures of the migrations created by the schedule will also result in events being logged against the object. These events can be seen by running a `kubectl describe` on the object

The output of `kubectl describe` will also show the status of the migrations that were triggered for each of the policies along with the start and finish times. The statuses will be maintained for the last successful migration and any Failed or InProgress migrations for each policy type.

Let's now run `kubectl describe` and see how the output would look like:

```text
kubectl describe migrationschedules.stork.libopenstorage.org -n mysql
```

```output
Name:         mysqlmigrationschedule

Namespace:    mysql
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"stork.libopenstorage.org/v1alpha1","kind":"MigrationSchedule","metadata":{"annotations":{},"name":"mysqlmigrationschedule",...
API Version:  stork.libopenstorage.org/v1alpha1
Kind:         MigrationSchedule
Metadata:
  Creation Timestamp:  2019-02-14T04:53:58Z
  Generation:          1
  Resource Version:    30206628
  Self Link:           /apis/stork.libopenstorage.org/v1alpha1/namespaces/mysql/migrationschedules/mysqlmigrationschedule
  UID:                 8a245c1d-3014-11e9-8d3e-0214683e8447
Spec:
  Schedule Policy Name:  daily
  Template:
    Spec:
      Cluster Pair:       remotecluster
      Include Resources:  true
      Namespaces:
        mysql
      Post Exec Rule:
      Pre Exec Rule:
      Selectors:           <nil>
      Start Applications:  false
Status:
  Items:
    Daily:
      Creation Timestamp:  2019-02-14T22:16:51Z
      Finish Timestamp:    2019-02-14T22:19:51Z
      Name:                mysqlmigrationschedule-daily-2019-02-14-221651
      Status:              Successful
    Interval:
      Creation Timestamp:  2019-02-16T00:40:52Z
      Finish Timestamp:    2019-02-16T00:41:52Z
      Name:                mysqlmigrationschedule-interval-2019-02-16-004052
      Status:              Successful
      Creation Timestamp:  2019-02-16T00:41:52Z
      Finish Timestamp:    <nil>
      Name:                mysqlmigrationschedule-interval-2019-02-16-004152
      Status:              InProgress
    Monthly:
      Creation Timestamp:  2019-02-14T20:05:41Z
      Finish Timestamp:    2019-02-14T20:07:41Z
      Name:                mysqlmigrationschedule-monthly-2019-02-14-200541
      Status:              Successful
    Weekly:
      Creation Timestamp:  2019-02-14T22:13:51Z
      Finish Timestamp:    2019-02-14T22:16:51Z
      Name:                mysqlmigrationschedule-weekly-2019-02-14-221351
      Status:              Successful
Events:
  Type    Reason      Age                    From   Message
  ----    ------      ----                   ----   -------
  Normal  Successful  4m55s (x53 over 164m)  stork  (combined from similar events): Scheduled migration (mysqlmigrationschedule-interval-2019-02-16-003652) completed successfully
```

Each migration is associated with a Migrations object. To get the most important information, type:

```text
kubectl get migration -n mysql
```

```output
NAME AGE
mysqlmigrationschedule-daily-2019-02-14-221651 1d
mysqlmigrationschedule-interval-2019-02-16-004052 5m
mysqlmigrationschedule-interval-2019-02-16-004152 4m
mysqlmigrationschedule-monthly-2019-02-14-200541 1d
mysqlmigrationschedule-weekly-2019-02-14-221351 1d
```

Once the MigrationSchedule object is deleted, all the associated Migration objects should be deleted as well.


### Failover an application

For instructions on how to failover an application, follow the steps from Metro DR to [Stop the application on the source cluster](/operations/operate-kubernetes/disaster-recovery/px-metro/4-failover-app/#stop-the-application-on-the-source-cluster-if-accessible) and then [Start the application on the destination cluster](/operations/operate-kubernetes/disaster-recovery/px-metro/4-failover-app/#start-the-application-on-the-destination-cluster).


## Clean up disaster recovery objects

If you no longer require a disaster recovery object, you can delete it.

To delete a migration schedule, run the following command:

```text
kubectl delete migrationschedule mysqlmigrationschedule -n mysql
```

To delete a `namespaced` policy, run the following command:

```text
kubectl delete namespacedschedulepolicy testpolicy -n mysql
```

To delete a cluster pair, run the following command:

```text
kubectl delete clusterpair remotecluster -n mysql
```

## Supported Kubernetes Resources
The Asynchronous DR feature supports the following Kubernetes resources:

* PersistentVolumeClaim
* PersistentVolume
* Deployment
* DeploymentConfig
* StatefulSet
* ConfigMap
* Service
* Secret
* DaemonSet
* ServiceAccount
* Role
* RoleBinding
* ClusterRole
* ClusterRoleBinding
* ImageStream
* Ingress
* Route
* Template
* CronJob
* ResourceQuota
* ReplicaSet
* LimitRange
* PodDisruptionBudget
* NetworkPolicy

  {{<info>}}**NOTE:** By default, Stork skips migrating NetworkPolicies which have `CIDR` set. To migrate NetworkPolicies which have `CIDR` set, use the `skipNetworkPolicyCheck: true` flag in the Migration object. {{</info>}}


Asynchronous DR also supports the following CRDs out-of-the-box:

* CassandraDatacenter
* CouchbaseBucket
* CouchbaseCluster
* CouchbaseEphemeralBucket
* CouchbaseMemcachedBucket
* CouchbaseReplication
* CouchbaseUser
* CouchbaseGroup
* CouchbaseRoleBinding
* CouchbaseBackup
* CouchbaseBackupRestore
* IBPCA
* IBPConsole
* IBPOrderer
* IBPPeer
* RedisEnterpriseCluster
* RedisEnterpriseDatabase


## Related videos

### Set up cross-cloud application migration from Google Cloud to Microsoft Azure with Portworx on OpenShift 4.2

{{< youtube TV6WAUqdzFI >}}

<br>

### Set up multi-cloud disaster recovery with Rancher Kubernetes Engine and Portworx

{{< youtube JClxvmc31j0 >}}

<br>

### Asynchronous DR for Postgres on Rancher Kubernetes Engine

{{< youtube id="4nV_A82VuwY" time="425" >}}

<br>

### Set up multi-cloud and multi-cluster data movement with Portworx on Openshift

{{< youtube 8aHuFTuByAM >}}
