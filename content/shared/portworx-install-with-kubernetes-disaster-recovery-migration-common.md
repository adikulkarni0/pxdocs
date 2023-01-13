---
title: Kubernetes disaster recovery and migration docs - migration common
description: Kubernetes disaster recovery and migration docs - migration common
keywords: Disaster recovery, Migration, kubernetes, k8s
hidden: true
---

### Troubleshooting

If there is a failure or you want more information about what resources were migrated you can `describe` the migration object using `kubectl`:

```text
kubectl describe migration <migrationschedulename> -n <migrationnamespace>
```
```output
Name:         zoomigrationschedule-interval-2022-12-12-220239
Namespace:    zookeeper
Labels:       <none>
Annotations:  stork.libopenstorage.org/migration-schedule-name: zoomigrationschedule
API Version:  stork.libopenstorage.org/v1alpha1
Kind:         Migration
Metadata:
  Creation Timestamp:  2022-12-12T22:02:39Z
  Finalizers:
    stork.libopenstorage.org/finalizer-cleanup
  Generation:  202
  Managed Fields:
    API Version:  stork.libopenstorage.org/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
    ......
    ...........
    Manager:    stork
    Operation:  Update
    Time:       2022-12-12T22:02:40Z
  Owner References:
    API Version:     stork.libopenstorage.org/v1alpha1
    Kind:            MigrationSchedule
    Name:            zoomigrationschedule
    UID:             5ae0cdb9-e4ba-4562-8094-8d7763d99ed1
  Resource Version:  31906052
  UID:               89c0ae5e-f08a-454f-b7b3-59951662ab0b
Spec:
  Admin Cluster Pair:                
  Cluster Pair:                      remotecluster
  Include Network Policy With CIDR:  false
  Include Optional Resource Types:   <nil>
  Include Resources:                 true
  Include Volumes:                   true
  Namespaces:
    zookeeper
  Post Exec Rule:           
  Pre Exec Rule:            
  Purge Deleted Resources:  false
  Selectors:                <nil>
  Skip Deleted Namespaces:  <nil>
  Skip Service Update:      false
  Start Applications:       false
Status:
  Finish Timestamp:                     2022-12-12T22:04:40Z
  Resource Migration Finish Timestamp:  2022-12-12T22:04:40Z
  Resources:
    Group:      core
    Kind:       PersistentVolume
    Name:       pvc-0a330e8b-d5f3-4c65-bca6-3eb37adcfb2d
    Namespace:  
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      core
    Kind:       PersistentVolume
    Name:       pvc-62ad252d-2218-4d33-9c57-8e61641b1a8c
    Namespace:  
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      core
    Kind:       PersistentVolume
    Name:       pvc-fdd559fa-b42c-4424-9699-cf13bc14a77e
    Namespace:  
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      core
    Kind:       PersistentVolumeClaim
    Name:       datadir-zk-0
    Namespace:  zookeeper
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      core
    Kind:       PersistentVolumeClaim
    Name:       datadir-zk-1
    Namespace:  zookeeper
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      core
    Kind:       PersistentVolumeClaim
    Name:       datadir-zk-2
    Namespace:  zookeeper
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      core
    Kind:       Service
    Name:       zk-cs
    Namespace:  zookeeper
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      core
    Kind:       Service
    Name:       zk-hs
    Namespace:  zookeeper
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      apps
    Kind:       StatefulSet
    Name:       zk
    Namespace:  zookeeper
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      policy
    Kind:       PodDisruptionBudget
    Name:       zk-pdb
    Namespace:  zookeeper
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
  Stage:        Final
  Status:       Successful
  Summary:
    Elapsed Time For Resource Migration:  7s
    Elapsed Time For Volume Migration:    1m54s
    Num Of Migrated Resources:            10
    Num Of Migrated Volumes:              3
    Total Bytes Migrated:                 12288
    Total Number Of Resources:            10
    Total Number Of Volumes:              3
  Volume Migration Finish Timestamp:      2022-12-12T22:04:33Z
  Volumes:
    Bytes Total:              4096
    Namespace:                zookeeper
    Persistent Volume Claim:  datadir-zk-0
    Reason:                   Migration successful for volume
    Status:                   Successful
    Volume:                   pvc-0a330e8b-d5f3-4c65-bca6-3eb37adcfb2d
    Bytes Total:              4096
    Namespace:                zookeeper
    Persistent Volume Claim:  datadir-zk-1
    Reason:                   Migration successful for volume
    Status:                   Successful
    Volume:                   pvc-62ad252d-2218-4d33-9c57-8e61641b1a8c
    Bytes Total:              4096
    Namespace:                zookeeper
    Persistent Volume Claim:  datadir-zk-2
    Reason:                   Migration successful for volume
    Status:                   Successful
    Volume:                   pvc-fdd559fa-b42c-4424-9699-cf13bc14a77e
Events:
  Type     Reason      Age                 From   Message
  ----     ------      ----                ----   -------
  Warning  Failed      20m                 stork  Error migrating volumes: Operation cannot be fulfilled on migrations.stork.libopenstorage.org "zoomigrationschedule-interval-2022-12-12-220239": the object has been modified; please apply your changes to the latest version and try again
  Normal   Successful  19m (x13 over 19m)  stork  Volume pvc-0a330e8b-d5f3-4c65-bca6-3eb37adcfb2d migrated successfully
  Normal   Successful  19m (x11 over 19m)  stork  Volume pvc-62ad252d-2218-4d33-9c57-8e61641b1a8c migrated successfully
```

## Pre and Post Exec rules

Similar to snapshots, a PreExec and PostExec rule can be specified when creating a Migration object. This will result in the PreExec rule being run before the migration is triggered and the PostExec rule to be run after the Migration has been triggered. If the rules do not exist, the Migration will log an event and will stop.

If the **PreExec rule fails** for any reason, it will log an event against the object and retry. **The Migration will not be marked as failed.**

If the **PostExec rule fails** for any reason, it will log an event and **mark the Migration as failed**. It will also try to cancel the migration that was started from the underlying storage driver.

In the example below with `mysql`, to add pre and post rules to our migration, we could edit our `migration.yaml` file like this:

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: Migration
metadata:
  name: mysqlmigration
  namespace: mysql
spec:
  clusterPair: remotecluster
  includeResources: true
  startApplications: true
  preExecRule: mysql-pre-rule
  postExecRule: mysql-post-rule
  namespaces:
  - mysql
```

## Advanced Operations

* [Migrating to GKE](/operations/operate-kubernetes/migration/gke/)
* [Migrating to EKS](/operations/operate-kubernetes/migration/eks/)
* [Configuring a namespace as a cluster namespace](/operations/operate-kubernetes/migration/cluster-admin-namespace)
