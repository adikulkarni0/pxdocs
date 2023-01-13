---
title: "Set up a Cluster Admin namespace for migration"
linkTitle: Set up a Cluster Admin namespace
keywords: cloud, backup, restore, snapshot, DR, migration, 
description: How to designate a namespace as the cluster admin namespace
weight: 200
series: migrate-with-stork
aliases:
    - /portworx-install-with-kubernetes/migration/cluster-admin-namespace/
---
By default, you can only migrate namespaces in which the Migration object is created. This is to prevent any user from migrating namespaces for which they do not have access. By default, `kube-system` is the admin namespace, but you can also designate a different namespace as the admin namespace. This will allow an admin who has access to that namespace to migrate any namespace from the source cluster.

In the following example, we will set up `portworx` as the admin namespace for migration by adding it to the Stork deployment object and passing in an additional parameter to the Stork deployment.

For a Portworx Operator based installation, perform the following steps.

1. Run the following command to edit the Portworx cluster spec:

    ```text
    kubectl edit storagecluster <clusterspecname> -n kube-system
    ```

2. In the editor, update the arguments to the Stork container to specify the cluster admin namespace using the `admin-namespace: portworx` parameter:

    ```
    stork:
        args:
          admin-namespace: portworx
          webhook-controller: "true"
    ```

3. Save the changes and wait for all the Stork pods to be recreated and in a running state after applying the changes (making note of the `AGE` section):

    ```text
    kubectl get pods -l name=stork -n kube-system
    ```
    ```output
    NAME                     READY   STATUS    RESTARTS   AGE
    stork-6fdd7d567b-4w86q   1/1     Running   0          63s
    stork-6fdd7d567b-lj47r   1/1     Running   0          60s
    stork-6fdd7d567b-m6pk7   1/1     Running   0          59s
    ```
    