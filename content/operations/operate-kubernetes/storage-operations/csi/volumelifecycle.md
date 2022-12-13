---
title: Volume Lifecycle Basics
keywords: csi, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk
description: This explains at a high level what the Portworx CSI Driver as compared to the Portworx in-tree plugin
weight: 200
aliases:
    - /portworx-install-with-kubernetes/storage-operations/create-pvcs/create-shared-pvcs-csi/
    - /portworx-install-with-kubernetes/storage-operations/csi/volumelifecycle/
---

## Create and use persistent volumes

Create and use volumes with CSI by configuring specs you create for your storage class, PVC, and volumes.

In this example, we are using the `px-csi-db` storage class out of the box. Please refer to [CSI Enabled Storage Classes](/portworx-install-with-kubernetes/storage-operations/csi/storageclasses/) for a list of available CSI enabled storage classes offered by Portworx.

1. Create a PersistentVolumeClaim based on the `px-csi-db` StorageClass:

    ```text
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: px-mysql-pvc
    spec:
      storageClassName: {{<highlight>}}px-csi-db{{</highlight>}}
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
    ```

2. Create a volume by referencing the PVC you created. This example creates a MySQL deployment referencing the `px-mysql-pvc` PVC you created in the previous step:

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
          labels:
            app: mysql
            version: "1"
        spec:
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
              claimName: {{<highlight>}}px-mysql-pvc{{</highlight>}}
    ```

## Create sharedv4 CSI-enabled volumes

Create sharedv4 CSI-enabled volumes by performing the following steps.

1. Create a sharedv4 PVC by creating the following `shared-pvc.yaml` file:

    ```text
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: px-mysql-pvc
    spec:
      storageClassName: {{<highlight>}}px-csi-db{{</highlight>}}
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 2Gi
    ```

2. Apply the `shared-pvc.yaml` file:

    ```text
    kubectl apply -f shared-pvc.yaml
    ```

## Clone volumes with CSI

You can clone CSI-enabled volumes, duplicating both the volume and content within it.

1. Create a PVC that references the PVC you wish to clone, specifying the `dataSource` with the `kind` and `name` of the target PVC you wish to clone. The following spec creates a clone of the `px-mysql-pvc` PVC in a YAML file named `clonePVC.yaml`:

      ```text
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
         name: clone-of-px-mysql-pvc
      spec:
         storageClassName: px-csi-db
         accessModes:
           - ReadWriteOnce
         resources:
           requests:
             storage: 2Gi
         dataSource:
          kind: {{<highlight>}}PersistentVolumeClaim{{</highlight>}}
          name: {{<highlight>}}px-mysql-pvc{{</highlight>}}
      ```

2. Apply the `clonePVC.yaml` spec to create the clone:

      ```text
      kubectl apply -f clonePVC.yaml
      ```

## Migrate to CSI PVCs

Currently, you cannot migrate or convert PVCs created using the native Kubernetes driver to the CSI driver. However, this is not required, and both types of PVCs can co-exist on the same cluster.


## Contribute

{{<companyName>}} welcomes contributions to its CSI implementation, which is open-source with a repository located at [OpenStorage](https://github.com/libopenstorage/openstorage). In addition, we also encourage contributions to the [Kubernetes-CSI open source implementation](https://github.com/kubernetes-csi).

