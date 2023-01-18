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

Portworx provides two types of sharedv4 features:

* Sharedv4 volumes
* Sharedv4 service volumes

## Prerequisites 

* Sharedv4 volumes must be enabled on your cluster. Portworx sharedv4 volumes are enabled by default.
* NFS ports [must be open](/operations/operate-kubernetes/storage-operations/create-pvcs/open-nfs-ports/).

## Provision a sharedv4 Volume

[Sharedv4 volumes](/concepts/shared-volumes/) are useful when you want multiple PODs to access the same PVC (volume) at the same time. They can use the same volume even if they are running on different hosts. They provide a global namespace and the semantics are POSIX compliant.

To increase fault tolerance, you can enable **sharedv4 service** volumes by [setting a value](#step-1-create-a-storageclass) for `sharedv4_svc_type`. With this feature enabled, every sharedv4 volume has a Kubernetes service associated with it. Sharedv4 service volumes expose the volume via a Kubernetes service IP. If the sharedv4 (NFS) server goes offline for a sharedv4 service volume and the volume requires a [failover](/concepts/shared-volumes/#sharedv4-failover-and-failover-strategy), only application pods that were running on the 2 nodes involved in failover need to be restarted.

{{<info>}}
**Notes about sharedv4 and sharedv4 service volumes:**

* A sharedv4 volume is created if and only if the PVC access mode is `ReadWriteMany` or `ReadOnlyMany`. The "sharedv4" setting in the storageClass does not matter. In other words, if an app expects a sharedv4 volume while using a ReadWriteOnce PVC, some of the pods may fail to start. The PVC will have to be modified to use ReadWriteMany or ReadOnlyMany access mode. 
* Sharedv4 service volumes are intended for use within the Kubernetes cluster where the volume resides.
* Sharedv4 service volumes default to using NFSv version 4.0.
* Sharedv4 (non-service) volumes default to using NFS version 3.
* Sharedv4 service volumes are not supported on Portworx clusters using Metro DR.
* On failover, applications may receive an error for non idempotent requests. For example, if an `mkdir` call is issued prior to failover, the client can resend it to the new server, which returns an `EEXIST` error if the directory was created by the first call.
{{</info>}}

### Step 1: Create a StorageClass


1. Create the following `portworx-sharedv4-sc.yaml` StorageClass, specifying your own values for the following fields:

  * The `metadata.name` field with a name for your StorageClass
  * The `parameters.repl` field with the replication factor you'd like to set
  * The `sharedv4` field set to `true`
  * (Optional) The `sharedv4_svc_type` field set to `"ClusterIP"` if you want to enable the sharedv4 service feature
  * (Optional) The `stork.libopenstorage.org/preferRemoteNodeOnly` field set to `"true"` if you want to strictly enforce [pod anti-hyperconvergence](/concepts/shared-volumes/#sharedv4-service-pod-anti-hyperconvergence) with respect to volume replica.
  * (Optional) The `sharedv4_failover_strategy` field set to `normal` or `aggressive` (shorter failover grace period)
      {{<info>}}
  **NOTE**: The default value for `sharedv4_failover_strategy` in sharedv4 volumes is `normal`, and the default value for `sharedv4_failover_strategy` in sharedv4 service volumes is `aggressive`.
      {{</info>}}

    ```text
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: portworx-rwx-rep2
    provisioner: pxd.portworx.com
    parameters:
      repl: "2"
      sharedv4: "true"
      sharedv4_svc_type: "ClusterIP"
    reclaimPolicy: Retain
    allowVolumeExpansion: true
    ```


2. Apply the StorageClass you created by running the following command:

    ```text
    kubectl apply -f portworx-sharedv4-sc.yaml
    ```

3. Verify that the StorageClass is created:

    ```text
    kubectl describe storageclass portworx-rwx-rep2
    ```
    ```output
    Name:            portworx-rwx-rep2
    IsDefaultClass:  No
    Annotations:     kubectl.kubernetes.io/last-applied-configuration={"allowVolumeExpansion":true,"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"portworx-rwx-rep2"},"parameters":{"disable_io_profile_protection":"1","io_profile":"auto","priority_io":"high","repl":"2","sharedv4":"true","sharedv4_svc_type":"ClusterIP"},"provisioner":"pxd.portworx.com","reclaimPolicy":"Retain"}
    Provisioner:           pxd.portworx.com
    Parameters:            disable_io_profile_protection=1,io_profile=auto,priority_io=high,repl=2,sharedv4=true,sharedv4_svc_type=ClusterIP
    AllowVolumeExpansion:  True
    MountOptions:          <none>
    ReclaimPolicy:         Retain
    VolumeBindingMode:     Immediate
    Events:                <none>
    ```

### Step 2: Create a persistent volume claim

1. Create a ReadWriteMany persistent volume claim. Save the following content into a file:

    ```text
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: px-sharedv4-pvc
      annotations:
        volume.beta.kubernetes.io/storage-class: portworx-rwx-rep2
    spec:
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 10Gi
    ```

2. Apply the spec:

    ```text
    kubectl create -f px-sharedv4-pvc.yaml
    ```

    Note that `accessModes` for this PVC is set to `ReadWriteMany` (RWX) so Kubernetes allows mounting this PVC on multiple pods.

3. Verify that the persistent volume claim is created:

    ```text
    kubectl get pvc
    ```

    ```output
    NAME              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     AGE
    px-sharedv4-pvc   Bound    pvc-9fdcc017-d4ed-40a1-811f-78dbc2ef7aeb   10Gi       RWX            portworx-rwx-rep2   46s
    ```

### Step 3: Create pods which use the persistent volume claim

We will start two pods which use the same shared volume.

1. Save the next two blocks into files pod1.yaml and pod2.yaml; use `kubectl apply -f pod1.yaml` and `kubectl apply -f pod2.yaml`

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

2. Verify that the pods are running:

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

1. Detach the volume by scaling the application pods down to 0.
2. Run the following `pxctl` command:

  ```text
  pxctl volume update --sharedv4_service_type=ClusterIP <volume>
  ```

3. Scale the deployment back up to start the application.

## Convert a sharedv4 service volume to a sharedv4 volume

Perform the following steps to convert a sharedv4 service volume to a sharedv4 volume:

1. Detach the volume by scaling the application pods down to 0.
2. Run the following `pxctl` command:

  ```text
  pxctl volume update --sharedv4_service_type="" <volume>
  ```
3. Scale the deployment back up to start the application.


## Convert an existing sharedv4 service volume to use preferRemoteNodeOnly

1. Scale down the application pods.

2. Run the following command to convert the volume to use `preferRemoteNodeOnly`:

    ```
    pxctl volume update --label stork.libopenstorage.org/preferRemoteNodeOnly="true" <volume>
    ```

3. Scale the pods back up to start the application.

## Update a legacy shared volume to a sharedv4 volume

You can convert an existing shared volume (deprecated) to a sharedv4 volume. Run the following command to update the volume setting:

```text
pxctl volume update --sharedv4=on <volume>
```

{{<info>}}To access PV/PVCs with a non-root user, refer [here](/operations/operate-kubernetes/storage-operations/create-pvcs/access-via-non-root-users).
{{</info>}}
