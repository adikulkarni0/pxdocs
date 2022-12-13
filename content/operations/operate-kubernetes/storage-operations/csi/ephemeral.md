---
title: Use Ephemeral volumes
keywords: csi, ephemeral, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk
description: This explains how to use ephemeral volumes with the Portworx CSI Driver
weight: 500
aliases:
    - /portworx-install-with-kubernetes/storage-operations/csi/ephemeral/
---
## Generic Ephemeral volumes

{{<info>}}
**NOTE:**
 FEATURE STATE: Kubernetes v1.23+ [stable]
{{</info>}}

Generic ephemeral volumes also work with typical storage operations such as snapshotting, cloning, resizing, and storage capacity tracking.

The following steps will allow you to create a generic ephemeral volume with the Portworx CSI Driver.

In this example we are using the `px-csi-db` storage class out of the box. Please refer [CSI Enabled Storage Classes](/portworx-install-with-kubernetes/storage-operations/csi/storageclasses/) for a list of available CSI enabled storage classes offered by Portworx.


1. Create a pod spec that uses a Portworx CSI Driver StorageClass, declaring the ephemeral volume as seen below in a yaml file named `ephemeral-volume-pod.yaml`:

      ```text
      kind: Pod
      apiVersion: v1
      metadata:
        name: my-app
      spec:
        containers:
          - name: my-frontend
            image: busybox
            volumeMounts:
            - mountPath: "/scratch"
              name: scratch-volume
            command: [ "sleep", "1000000" ]
        volumes:
          - name: scratch-volume
            ephemeral:
              volumeClaimTemplate:
                metadata:
                  labels:
                    type: my-frontend-volume
                spec:
                  accessModes: [ "ReadWriteOnce" ]
                  storageClassName: "px-csi-db"
                  resources:
                    requests:
                      storage: 1Gi
      ```

2. Apply the `ephemeral-volume-pod.yaml` spec to create the pod with a generic ephemeral volume:

      ```text
      kubectl apply -f ephemeral-volume-pod.yaml
      ```

## Migration to CSI PVCs

Currently, you cannot migrate or convert PVCs created using the native Kubernetes driver to the CSI driver. However, this is not required, and both types of PVCs can co-exist on the same cluster.


## Contribute

{{<companyName>}} welcomes contributions to its CSI implementation, which is open-source with a repository located at [OpenStorage](https://github.com/libopenstorage/openstorage). In addition, we also encourage contributions to the [Kubernetes-CSI open source implementation](https://github.com/kubernetes-csi).
