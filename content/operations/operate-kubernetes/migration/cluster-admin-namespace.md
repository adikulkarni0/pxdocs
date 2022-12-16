---
title: "Set up a Cluster Admin namespace for Migration"
linkTitle: Set up a Cluster Admin namespace
keywords: cloud, backup, restore, snapshot, DR, migration, 
description: How to designate a namespace as the cluster admin namespace
weight: 100
aliases:
    - /portworx-install-with-kubernetes/migration/cluster-admin-namespace/
---
By default, you can only migrate namespaces in which the `ClusterPair`, `MigrationSchedule`, and `migration` are created.
This is to prevent any user from migrating namespaces for which they do not have access. 

You can also designate one namespace as an admin namespace, which allows an admin who has access to that namespace to migrate any namespace from the source cluster. This requires passing in the `admin-namespace` argument to the `StorageCluster`, and you also must have created the `ClusterPair`, `MigrationSchedule`, and `migration` in the admin namespace. By default, `kube-system` namespace is designated as an admin namespace. To learn how to create `ClusterPair`, `MigrationSchedule`, and `migration` objects in `admin namespace`, refer to [migration with Stork on Kubernetes](/operations/operate-kubernetes/migration/).

Run the following command to edit the `StorageCluster`. In the editor, update the arguments to `StorageCluster` (under `spec.stork.args`) to specify the cluster `admin-namespace` argument. For example:

```text
kubectl edit stc <StorageCluster-name> -n kube-system
```

```text
stork:
      args:
        admin-namespace: {{<highlight>}}<admin-namespace>{{</highlight>}}
        webhook-controller: "true"
      enabled: true
```

Save the changes and wait for all the Stork pods to be in running state after applying the changes:

```text
kubectl get pods -n kube-system -l name=stork
```
{{<info>}}**NOTE:** You can add the above arguments to the `StorageCluster` spec if you need to configure the admin namespace as part of new Portworx installation.{{</info>}}
