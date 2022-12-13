---
title: Portworx with CSI
keywords: csi, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk
description: Portworx CSI Driver landing page
series: k8s-storage
aliases:
    - /portworx-install-with-kubernetes/storage-operations/csi/
---
[CSI](https://kubernetes-csi.github.io/), or _Container Storage Interface_, is a model for integrating storage system service with Kubernetes and other orchestration systems. Kubernetes has supported CSI since 1.10 as beta.

With CSI, Kubernetes gives storage drivers the opportunity to release on their schedule. This allows storage vendors to upgrade, update, and enhance their drivers without the need to update Kubernetes, maintaining a consistent, dependable, orchestration system.

Using Portworx with CSI, you can perform the following operations:

* Create and use CSI-enabled persistent volumes
* Secure your CSI-enabled volumes with token authorization and encryption defined at the StorageClass or the PVC level
* Take snapshots of CSI-enabled volumes
* Create sharedv4 CSI-enabled volumes

## Supported features

The following table shows the core features supported by CSI and which minimum versions of Portworx and Kubernetes they require. 

| Feature | Portworx version | Supported Kubernetes version |
| --- | --- | --- |
| [Provision, attach, and mount volumes](/operations/operate-kubernetes/storage-operations/csi/volumelifecycle) | 2.2 | 1.13.12 |
| [CSI Snapshots](/operations/operate-kubernetes/storage-operations/csi/dataprotection) | 2.2 | 1.17 |
| Stork [^1] | 2.2 | 1.13 |
| Volume Expansion (resizing) | 2.2 | 1.16 |
| [Generic Ephemeral Volumes](/operations/operate-kubernetes/storage-operations/csi/ephemeral) | 2.2 | 1.21 |
| [CSI Raw Block](/operations/operate-kubernetes/storage-operations/csi/rawblock) | 2.8 | 1.14 |

{{<companyName>}} does not recommend that you use alpha Kubernetes features in production as the API and core functionality are not finalized. Users that adopt alpha features in production may need to perform costly manual upgrades.

[^1]: Note that only Stork 2.3.0 or later is supported with CSI.

## Contribute

{{<companyName>}} welcomes contributions to its CSI implementation, which is open-source with a repository located at [OpenStorage](https://github.com/libopenstorage/openstorage). In addition, we also encourage contributions to the [Kubernetes-CSI open source implementation](https://github.com/kubernetes-csi).
