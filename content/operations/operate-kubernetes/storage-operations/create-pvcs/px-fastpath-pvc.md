---
title: Create PX-Fast PVCs
description: How to enable PX-Fast functionality on volumes
keywords: portworx, kurbernetes, containers, storage, PX-Fast, PX-StoreV2
series: k8s-vol
disableprevnext: true
weight: 300
---

PX-Fast is a Portworx feature that enables an accelerated IO path for the volumes that meet certain prerequisites. It is optimized for workloads requiring consistent low latencies. PX-Fast is built on top of a Portworx PX-StoreV2 datastore. 

{{<info>}}**NOTE:** 

* PX-Fast requires a special license for the functionality to be active. Contact [Portworx support](/support) team for obtaining this license. 

* PX-Fast works only if Portworx is installed with PX-StoreV2 datastore.{{</info>}}

## Prerequisites 

You must have deployed [Portworx with PX-StoreV2 datastore](/install-portworx/on-premises/install-px-store-v2).

## Create PX-Fast PVCs 

Perform the following steps to create PX-Fast volumes.

### Create a StorageClass

1. Save the following as a YAML file:

    ```text
    kind: StorageClass
	apiVersion: storage.k8s.io/v1
	metadata:
  		name: px-fast
	provisioner: kubernetes.io/portworx-volume
	parameters:
  		repl: "1"
  		fastpath: "true"
	allowVolumeExpansion: true
    ```
	Note that this StorageClass will have 1 replica, and the Portworx volumes referring to this StorageClass will also have 1 replica.

2. Run the following `kubectl apply` command to create a StorageClass:

    ```text
    kubectl apply -f <px-fast>.yaml
    ```
    ```output
	storageclass.storage.k8s.io/px-fast-sc created
   	```
3. Verify that the StorageClass is created:

	```text
	kubectl describe storageclass px-fast-sc
	```
	```output
	Provisioner:           kubernetes.io/portworx-volume
	Parameters:            fastpath=true,repl=1
	AllowVolumeExpansion:  <unset>
	MountOptions:          <none>
	ReclaimPolicy:         Delete
	VolumeBindingMode:     Immediate
	Events:                <none>
	```
### Create a PVC

1. Save the following as YAML file:

	```text
	kind: PersistentVolumeClaim
	apiVersion: v1
	metadata:
  		name: px-fast-pvc
	spec:
  		storageClassName: px-fast
  		accessModes:
    		- ReadWriteOnce
  		resources:
    		requests:
      			storage: 1Gi
	```

2. Run the following `kubectl apply` command to create a PX-Fast PVC:

	```text 
	kubectl apply -f <px-pvc>.yaml
	```

3. Verify that a persistent volume claim is created:

	```text
	kubectl get pvc
	```

### Deploy an application using a PX-Fast PVC

To utilize PX-Fast on the above PVCs, the application pods must run on the same node where the volume replica exists. The following sample deployment spec uses the `stork.libopenstorage.org/preferLocalNodeOnly: "true"` annotation, which enforces the pod to be scheduled on the node where the volume replica exists:

```text
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  replicas: 1
  template:
    metadata:
      annotations:
        stork.libopenstorage.org/preferLocalNodeOnly: "true"
      labels:
        app: mysql
    spec:
      schedulerName: stork
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: px-fast-pvc
```

Note that the `schedulerName`field in the above spec is set to `stork`. From Stork version 2.9.0 or newer, the `schedulerName` is set to `stork` by default. 

The annotation `stork.libopenstorage.org/preferLocalNodeOnly: "true"` enforces the pod to be scheduled on the node where the volume replica exists.  To avoid this enforced behavior, you can choose not to specify the `stork.libopenstorage.org/preferLocalNodeOnly: true` annotation. In such a case, Stork will do best-effort to place the application pod on the node where the replica exists, but if the application pod gets attached to a non-replica node, it cannot use PX-Fast.

{{<info>}}**NOTE:** PX-Fast is not supported for PVCs with more than 1 replica. It is also not supported on the aggregated volumes.{{</info>}}



## Reference

[PX-Fast volumes using pxctl](/reference/cli/px-fast-cli)


