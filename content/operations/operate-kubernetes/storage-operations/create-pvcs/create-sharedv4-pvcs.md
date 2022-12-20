---
title: Create sharedv4 PVCs
weight: 300
keywords: sharedv4 volumes, ReadWriteMany, PVC, kubernetes, k8s
description: Learn how to use Portworx sharedv4 volumes (ReadWriteMany) in your Kubernetes cluster.
series: k8s-vol
aliases:
    - /portworx-install-with-kubernetes/storage-operations/create-pvcs/create-shared-pvcs/
    - /portworx-install-with-kubernetes/storage-operations/create-pvcs/create-sharedv4-pvcs/
---

This document describes how to use Portworx **sharedv4** (ReadWriteMany) volumes in your Kubernetes cluster.

## Prerequisites

* sharedv4 volumes must not be disabled on your cluster.
* NFS ports between nodes [must be open](/operations/operate-kubernetes/storage-operations/create-pvcs/open-nfs-ports/).

## Provision a Sharedv4 Volume

[Sharedv4 volumes](/concepts/shared-volumes/) are useful when you want multiple PODs to access the same PVC (volume) at the same time. They can use the same volume even if they are running on different hosts. They provide a global namespace and the semantics are POSIX compliant.

To increase fault tolerance, you can enable **Sharedv4 service** volumes by [setting a value](#step-1-create-a-storageclass) for `sharedv4_svc_type`. With this feature enabled, every sharedv4 volume has a Kubernetes service associated with it. Sharedv4 service volumes expose the volume via a Kubernetes service IP. If the sharedv4 (NFS) server goes offline for a sharedv4 service volume and the volume requires a [failover](/concepts/shared-volumes/#sharedv4-failover-and-failover-strategy), only application pods that were running on the 2 nodes involved in failover need to be restarted.

{{<info>}}
**Notes about Sharedv4 service volumes:**

* Sharedv4 service volumes are intended for use within the Kubernetes cluster where the volume resides.
* Sharedv4 service volumes default to using NFSv4.0.
* <u>Known issues</u>: 
  * On failover, Applications may receive an error for non idempotent requests. For example, if an `mkdir` call is issued prior to failover, the client can resend it to the new server, which returns an `EEXIST` error if the directory was created by the first call.
* Sharedv4 service volumes are not supported on Portworx clusters setup in Metro DR deployment model.
{{</info>}}

### Step 1: Create a StorageClass

1. Create the following StorageClass, specifying your own values for the following fields:

  * The `metadata.name` field with a name for your storageClass
  * The `parameters.repl` field with the replication factor you'd like to set
  * The `sharedv4` field set to `true`
  * (Optional) The `sharedv4_svc_type` field set to `"ClusterIP"` if you want to enable the sharedv4 service feature
  * (Optional) The `stork.libopenstorage.org/preferRemoteNodeOnly` field set to `"true"` if you want to strictly enforce [pod anti-hyperconvergence](/concepts/shared-volumes/#sharedv4-service-pod-anti-hyperconvergence) with respect to volume replica.
  * (Optional) The `sharedv4_failover_strategy` field set to `normal` or `aggressive` (shorter failover grace period)
{{<info>}}
  **NOTE**: The default value for `sharedv4_failover_strategy` in sharedv4 volumes is `normal`, and the default value for `sharedv4_failover_strategy` in sharedv4 service volumes is `aggressive`.
{{</info>}}

        ```text
        kind: StorageClass
        apiVersion: storage.k8s.io/v1
        metadata:
            name: px-sharedv4-sc
        provisioner: kubernetes.io/portworx-volume
        parameters:
           repl: "2"
           sharedv4: "true"
           sharedv4_svc_type: "ClusterIP"
           stork.libopenstorage.org/preferRemoteNodeOnly: "true"
        ```

2. Apply the StorageClass:

```text
kubectl apply -f examples/volumes/portworx/portworx-sharedv4-sc.yaml
```

3. Verify that the StorageClass is created:

```text
kubectl describe storageclass px-sharedv4-sc
```

```output
Name:        px-sharedv4-sc
IsDefaultClass:    No
Annotations:     <none>
Provisioner:     kubernetes.io/portworx-volume
Parameters:    repl=2,sharedv4=true,sharedv4_svc_type=ClusterIP
Events:     <none>
```

### Step 2: Create persistent volume claim

Create a ReadWriteMany persistent volume claim:

```text
kubectl create -f examples/volumes/portworx/portworx-volume-sharedv4-pvc.yaml
```

Example:

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: px-sharedv4-pvc
  annotations:
    volume.beta.kubernetes.io/storage-class: px-sharedv4-sc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```

Note that `accessModes` for this PVC is set to `ReadWriteMany` so the Kubernetes allows mounting this PVC on multiple pods.

Verify that the persistent volume claim is created:

```text
kubectl get pvc
```

```output
NAME            STATUS    VOLUME                                   CAPACITY   ACCESSMODES   STORAGECLASS   AGE
px-sharedv4-pvc   Bound     pvc-a38996b3-76e9-11e7-9d47-080027b25cdf 10Gi       RWX           px-sharedv4-sc   12m

```

### Step 3: Create pods which use the persistent volume claim

We will start two pods which use the same shared volume.

Starting pod-1

```text
kubectl create -f examples/volumes/portworx/portworx-volume-sharedv4-pod-1.yaml
```

Example:

```text
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: test-container
    image: gcr.io/google_containers/test-webserver
    volumeMounts:
    - name: test-volume
      mountPath: /test-portworx-volume
  volumes:
  - name: test-volume
    persistentVolumeClaim:
      claimName: px-sharedv4-pvc
```

Starting pod-2

```text
kubectl create -f examples/volumes/portworx/portworx-volume-sharedv4-pod-2.yaml
```

Example:

```text
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - name: test-container
    image: gcr.io/google_containers/test-webserver
    volumeMounts:
    - name: test-volume
      mountPath: /test-portworx-volume
  volumes:
  - name: test-volume
    persistentVolumeClaim:
      claimName: px-sharedv4-pvc
```

Verifying pods are running:

```text
kubectl get pods
```

```output
NAME      READY     STATUS    RESTARTS   AGE
pod1      1/1       Running   0          2m
pod2      1/1       Running   0          1m
```

## Convert a sharedv4 volume to a sharedv4 service volume

Perform the following steps to convert a sharedv4 volume to use the new sharedv4 service feature:

1. Detach the volume by scaling down the application pods down to 0.
1. Run the following `pxctl` command:

  ```text
  pxctl volume update --sharedv4_service_type=ClusterIP <volume>
  ```

1. Scale the pods back up to start the application.

## Convert a sharedv4 service volume to a sharedv4 volume


Perform the following steps to convert a sharedv4 service volume to a sharedv4 volume:

1. Detach the volume by scaling the application pods down to 0.
1. Run the following `pxctl` command:

  ```text
  pxctl volume update --sharedv4_service_type="" <volume>
  ```
1. Scale the pods back up to start the application.

## Convert an existing sharedv4 service volume to use preferRemoteNodeOnly

1. Scale down the application pods.

2. Run the following command to convert the volume to use `preferRemoteNodeOnly`:

    ```
    pxctl volume update --label stork.libopenstorage.org/preferRemoteNodeOnly="true" <volume>
    ```

3. Scale the pods back up to start the application.

## Update a legacy shared volume to a sharedv4 volume

You can update an existing shared volume to use the new v4 protocol and convert it into a sharedv4 volume. Run the following pxctl command to update the volume setting

```text
pxctl volume update --sharedv4=on <volume>
```

{{<info>}}To access PV/PVCs with a non-root user, refer [here](/operations/operate-kubernetes/storage-operations/create-pvcs/access-via-non-root-users).
{{</info>}}
