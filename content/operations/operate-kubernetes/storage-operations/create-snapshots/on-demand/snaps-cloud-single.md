---
title: Cloud backups for single PVCs
weight: 200
hidden: true
keywords: cloud backup, single PVC, stork, cloudsnaps, kubernetes, k8s
description: Instructions for backing up a PVC with consistency to cloud and restore PVCs from the backup
series: k8s-cloud-snap
aliases:
    - /portworx-install-with-kubernetes/storage-operations/create-snapshots/on-demand/snaps-cloud-single/
---
This document will show you how to create cloud snapshots of Portworx volumes and how you can clone those snapshots to use them in pods.

## Pre-requisites

### Installing Stork

This requires that you already have [Stork] (/operations/operate-kubernetes/storage-operations/stork) installed and running on your
Kubernetes cluster. If you fetched the Portworx specs from the Portworx spec generator in [PX-Central](https://central.portworx.com) and used the default options, Stork is already installed.

### Portworx version

Cloud snapshots using below method is supported in Portworx version 1.4 and above.
Cloud snapshots (for aggregated volumes) using below method is supported in Portworx version 2.0 and above.

{{< content "shared/portworx-install-with-kubernetes-storage-operations-create-snapshots-k8s-cloud-snap-creds-prereq.md" >}}

## Creating cloud snapshots

The cloud snapshot method supports the following annotations:

* __portworx/snapshot-type__: Indicates the type of snapshot. For cloud snapshots, the value should be **cloud**.
* __portworx/cloud-cred-id__: (Optional) This specifies the credentials UUID if you have configured credentials for multiple cloud providers. In a situation where a single cloud provider is configured, this is not required.
* __portworx.io/cloudsnap-incremental-count__: This specifies the number of incremental cloud snapshots after which a full backup will be taken.

**Example**

In below example, we create a cloud snapshot for a PVC called _mysql-data_ backed by a Portworx volume.

```text
apiVersion: volumesnapshot.external-storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: mysql-snapshot
  namespace: default
  annotations:
    portworx/snapshot-type: cloud
spec:
  persistentVolumeClaimName: mysql-data
```

Once you apply the above object you can check the status of the snapshots using the `kubectl get volumesnapshot.volumesnapshot.external-storage.k8s.io/` command with the name of your snapshot appended:

```text
kubectl get volumesnapshot.volumesnapshot.external-storage.k8s.io/mysql-snapshot
```

```output
NAME                             AGE
volumesnapshots/mysql-snapshot   2s
```

```text
kubectl get volumesnapshotdatas
```

```output
NAME                                                                            AGE
volumesnapshotdatas/k8s-volume-snapshot-2bc36c2d-227f-11e8-a3d4-5a34ec89e61c    1s
```

The creation of the volumesnapshotdatas object indicates that the snapshot has been created. If you describe the volumesnapshotdatas object you can see the Portworx Cloud Snapshot ID and the PVC for which the snapshot was created.

```text
kubectl describe volumesnapshotdatas
```

```output
Name:         k8s-volume-snapshot-2bc36c2d-227f-11e8-a3d4-5a34ec89e61c
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  volumesnapshot.external-storage.k8s.io/v1
Kind:         VolumeSnapshotData
Metadata:
  Cluster Name:
  Creation Timestamp:             2018-03-08T03:17:02Z
  Deletion Grace Period Seconds:  <nil>
  Deletion Timestamp:             <nil>
  Resource Version:               29989636
  Self Link:                      /apis/volumesnapshot.external-storage.k8s.io/v1/k8s-volume-snapshot-2bc36c2d-227f-11e8-a3d4-5a34ec89e61c
  UID:                            2bc3a203-227f-11e8-98cc-0214683e8447
Spec:
  Persistent Volume Ref:
    Kind:  PersistentVolume
    Name:  pvc-f782bf5c-20e7-11e8-931d-0214683e8447
  Portworx Volume:
    Snapshot Id:  687c60d0-73cb-4055-a08f-33c5ab8d4d8e/149813028909420894-125009403033610837-incr
  Volume Snapshot Ref:
    Kind:  VolumeSnapshot
    Name:  default/mysql-snapshot-2b2150dd-227f-11e8-98cc-0214683e8447
Status:
  Conditions:
    Last Transition Time:  <nil>
    Message:
    Reason:
    Status:
    Type:
  Creation Timestamp:      <nil>
Events:                    <none>
```

## Creating PVCs from cloud snapshots

When you install Stork, it also creates a storage class called _stork-snapshot-sc_. This storage class can be used to create PVCs from snapshots.

To create a PVC from a snapshot, you would add the `snapshot.alpha.kubernetes.io/snapshot` annotation to refer to the snapshot
name.

For the above snapshot, the spec would like this:

```text
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-snap-clone
  annotations:
    snapshot.alpha.kubernetes.io/snapshot: mysql-snapshot
spec:
  accessModes:
     - ReadWriteOnce
  storageClassName: stork-snapshot-sc
  resources:
    requests:
      storage: 2Gi
```

Once you apply the above spec you will see a PVC created by Stork. This PVC will be backed by a Portworx volume clone of the snapshot created above.

```text
kubectl get pvc
```

```output
NAMESPACE   NAME                                   STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS                AGE
default     mysql-data                             Bound     pvc-f782bf5c-20e7-11e8-931d-0214683e8447   2Gi        RWO            px-mysql-sc                 2d
default     mysql-snap-clone                       Bound     pvc-05d3ce48-2280-11e8-98cc-0214683e8447   2Gi        RWO            stork-snapshot-sc           2s
```
