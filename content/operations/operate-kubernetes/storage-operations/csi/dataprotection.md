---
title: Data Protection and Snapshots
keywords: csi, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk, snapshot
description: This explains how you can protect your data against failures with Stork or CSI Snapshotting
weight: 300
aliases:
    - /portworx-install-with-kubernetes/storage-operations/csi/dataprotection/
---
## Setup CSI Volume Snapshotting 

In order to use VolumeSnapshots with the Portworx CSI Driver and Portworx Operator, you must enable Snapshot Controller in your StorageCluster. By default, `installSnapshotController` is set to `true` when you enable CSI in the StorageCluster.

Run the following command to edit the `StorageCluster` and update the arguments if CSI is not enabled:

```text
kubectl edit stc <StorageClustername> -n kube-system
```
```textspec:
    csi:
      enabled: true
      installSnapshotController: true
```

## Take snapshots of CSI-enabled volumes

For Kubernetes 1.20+, CSI Snapshotting is supported by the Portworx CSI Driver.

If you already have a [CSI PVC](/operations/operate-kubernetes/storage-operations/csi/volumelifecycle/#create-and-use-persistent-volumes), complete the following steps to create and restore a CSI VolumeSnapshot.

1. Create a VolumeSnapshotClass, specifying the following:

    * The `snapshot.storage.kubernetes.io/is-default-class: "true"` annotation
    * The `csi.storage.k8s.io/snapshotter-secret-name` parameter with your encryption and/or authorization secret
    * The `csi.storage.k8s.io/snapshotter-secret-namespace` parameter with the namespace your secret is in

    {{<info>}}**Note**:
Specify `snapshotter-secret-name` and `snapshotter-secret-namespace` if px-security is `ENABLED`. 

See [enable security in Portworx](/cloud-references/security/kubernetes/shared-secret-model-operator/) for more information.
    {{</info>}}

    ```text
    apiVersion: snapshot.storage.k8s.io/v1
    kind: VolumeSnapshotClass
    metadata:
      name: px-csi-snapclass
      annotations:
        snapshot.storage.kubernetes.io/is-default-class: "true"
    driver: pxd.portworx.com
    deletionPolicy: Delete
    parameters: ## Specify only if px-security is ENABLED
      csi.storage.k8s.io/snapshotter-secret-name: px-user-token  
      csi.storage.k8s.io/snapshotter-secret-namespace: kube-system 
    ```

2. Create a VolumeSnapshot:

   ```text  
   apiVersion: snapshot.storage.k8s.io/v1
   kind: VolumeSnapshot
   metadata:
     name: px-csi-snapshot
   spec:
     volumeSnapshotClassName: px-csi-snapclass
     source:
       persistentVolumeClaimName: px-mysql-pvc
   ```

   {{<info>}}**NOTE:** VolumeSnapshot objects are namespace-scoped and should be created in the same namespace as the PVC.{{</info>}}

3. Restore from a VolumeSnapshot:

   ```text
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: px-csi-pvc-restored 
   spec:
     storageClassName: px-csi-db
     dataSource:
       name: px-csi-snapshot
       kind: VolumeSnapshot
       apiGroup: snapshot.storage.k8s.io
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 2Gi
   ```

See the [Kubernetes-CSI snapshotting documentation](https://kubernetes-csi.github.io/docs/snapshot-restore-feature.html) for more examples and documentation. 

## Contribute

{{<companyName>}} welcomes contributions to its CSI implementation, which is open-source with a repository located at [OpenStorage](https://github.com/libopenstorage/openstorage). In addition, we also encourage contributions to the [Kubernetes-CSI open source implementation](https://github.com/kubernetes-csi).
