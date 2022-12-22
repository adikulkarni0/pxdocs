---
title: Clean up disaster recovery objects
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration
description: How to remove disaster recovery object.
weight: 600
---

If you no longer require a disaster recovery object, you can delete it.

Perform the following steps from a location where you have `kubectl` access to the source cluster:

1. Delete the migration schedule:

    ```text
    kubectl delete migrationschedules <migrationschedule-name> -n <migrationnamespace>
    ```
    {{<info>}}
**NOTE:** Once a `migrationschedule` object is deleted, delete all the associated migration objects that you retrieved in step 2 of the [previous section](/operations/operate-kubernetes/disaster-recovery/async-dr/schedule-migration/#check-your-migration-status-from-your-source-cluster).
    {{</info>}}


2. Delete the `namespaced` policy associated:

    ```text
    kubectl delete schedulepolicy <your-schedule-policy> -n <migrationnamespace>
    ```

3. Delete the cluster pair:

    ```text
    kubectl delete clusterpair <remoteclustername> -n <migrationnamespace>
    ```

    
    
   