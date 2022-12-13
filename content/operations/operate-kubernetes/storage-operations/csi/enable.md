---
title: Enable CSI
keywords: csi, install, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk
description: This explains at a high level what the Portworx CSI Driver as compared to the Portworx in-tree plugin
weight: 100
aliases:
    - /portworx-install-with-kubernetes/storage-operations/csi/enable/
---

## Prerequisites

Before you install and use Portworx with CSI, ensure you meet the prerequisistes:

* Openshift users must use Openshift 4.1 or higher
* Kubernetes users must use 1.13.12 or higher
* Certain features may require newer versions of Kubernetes or Portworx

## Overview

The Portworx CSI Driver requires additional components called `CSI Sidecars` in order to function properly. To make sure these components and their dependencies are installed on your cluster, follow the instructions below. 

## Portworx CSI installed by default

Starting with the Portworx [Operator 1.5 and higher](/reference/crd/storage-cluster/), CSI is enabled by default for new Portworx installations. Due to the nature of CSI and its dependencies, it is highly recommend to use the Portworx Operator instead of the spec generator for installation.

The Portworx Operator easily manages all CSI components based on your Kubernetes and Portworx versions. This makes upgrading Kubernetes and Portworx versions far easier.

{{<info>}}**Note**:
Explicitly disabling CSI will also prevent the Pure Storage FlashBade DirectAccess mode from working.
{{</info>}}

## Portworx Operator 1.8 and higher 

Starting with the Portworx [Operator 1.8 and higher](/reference/crd/storage-cluster/), new syntax is available for CSI, which is enabled by default for new Portworx installations. The default values are as follows:

```text
spec:
  csi:
    enabled: true
    installSnapshotController: false
```

### Migrations to Operator 1.8

If you migrate from an earlier version to Operator 1.8, your `spec` is automatically converted to match the new syntax. It will reflect the value you had previously set for `spec.featureGates.CSI`.

If you previously [enabled Snapshot Controller manually](/operations/operate-kubernetes/storage-operations/csi/dataprotection/#setup-csi-volume-snapshotting), your existing controller will still work but `spec.csi.installSnapshotController` will be set to `false`. If you wish to switch to a Snapshot Controller installation that is managed by Portworx, remove your Snapshot Controller deployment and set `spec.csi.installSnapshotController` equal to `true`.

## Openshift installation

If you are using Openshift, you must add the `px-csi-account` service account to the privileged security context.

```text
oc adm policy add-scc-to-user privileged system:serviceaccount:kube-system:px-csi-account
```


{{<info>}}**Note**:
If you experience the error
`errorUnable to update cluster as v1alpha1 version of ... Remove these CRDs to allow the upgrade to proceed`, 
follow [this solution from RedHat](https://access.redhat.com/solutions/5372561).
{{</info>}}


## Contribute

{{<companyName>}} welcomes contributions to its CSI implementation, which is open-source with a repository located at [OpenStorage](https://github.com/libopenstorage/openstorage). In addition, we also encourage contributions to the [Kubernetes-CSI open source implementation](https://github.com/kubernetes-csi).
