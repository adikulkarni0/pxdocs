---
title: Raw Block Devices
keywords: csi, install, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk, raw, block, device
description: This explains how to use raw block devices with the Portworx CSI Driver.
weight: 600
aliases:
    - /portworx-install-with-kubernetes/storage-operations/csi/rawblock/
---
The Portworx CSI Driver supports both File and Block volume types in Kubernetes PVCs. While file has more supported features, block support for `ReadWriteOnly` PVCs was added in Portworx Enterprise in version 2.8.0. 

## Use cases for Raw Block Devices

If your application has a requirement to consume a raw block device as opposed to a mounted filesystem, this is a perfect use case for Portworx CSI Raw Block devices.

Standard Portworx volumes are used when creating a Raw Block PVC, so all features (snapshotting, resizing, encryption, security, and so on) are all available out of the box for these types of volumes.

{{<info>}}**Note**:
Currently, only ReadWriteOnce PVCs can be created with the block volume mode.
{{</info>}}


## Create and use raw block PVCs

In this example we are using the `px-csi-db` storage class out of the box. Please refer [CSI Enabled Storage Classes](/portworx-install-with-kubernetes/storage-operations/csi/storageclasses/) for list of available CSI enabled storage classes offered by Portworx.

1. Create a PVC spec that references the portworx-csi-sc as seen below in a yaml file named `raw-block-pvc.yaml`:

     
    ```text
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
       name: px-csi-raw-block-pvc
    spec:
       volumeMode: Block
       storageClassName: px-csi-db
       accessModes:
         - ReadWriteOnce
       resources:
         requests:
           storage: 2Gi
    ```

2. Apply the `raw-block-pvc.yaml` spec to create the raw block PVC:

    ```text
    kubectl apply -f raw-block-pvc.yaml
    ```

3. Create a deployment SPEC that references the above raw block PVC in `raw-block-deployment.yaml`:

    ```text
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: ioping
    spec:
      selector:
        matchLabels:
          app: ioping
      strategy:
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 1
        type: RollingUpdate
      replicas: 1
      template:
        metadata:
          labels:
            app: ioping
            version: "1"
        spec:
          containers:
          - name: ioping 
            image: hpestorage/ioping 
            command: [ "ioping" ] 
            args: [ "/dev/xvda" ] 
            volumeDevices: 
            - name: raw-device 
              devicePath: /dev/xvda 
          volumes:
          - name: raw-device
            persistentVolumeClaim:
              claimName: px-csi-raw-block-pvc
    ```

4. Apply the `raw-block-deployment.yaml` spec to create the deployment utilizing our raw block PVC:

    ```text
    kubectl apply -f raw-block-deployment.yaml
    ```