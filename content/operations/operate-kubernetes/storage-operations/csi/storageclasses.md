---
title: CSI Enabled Storage Classes by Portworx
keywords: csi, install, portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk, storage class
description: This explains at a high level what the Portworx CSI Driver as compared to the Portworx in-tree plugin
weight: 100
aliases:
    - /portworx-install-with-kubernetes/storage-operations/csi/storageclasses/
---

## CSI Enabled Storage Classes

{{<companyName>}} offers a set of storage classes out of the box for various use cases. We recommend that you use any of the StorageClasses listed in the following output. Refer to [StorageClass Parameters](/operations/operate-kubernetes/storage-operations/create-pvcs/dynamic-provisioning) for more information about StorageClass parameters that Portworx supports.

{{<info>}}
**NOTE:**
Enable CSI for a StorageClass by setting the `provisioner` value to `pxd.portworx.com`.
{{</info>}}

```test
NAME                                 PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
px-csi-db                            pxd.portworx.com                Delete          Immediate           true                   12d
px-csi-db-cloud-snapshot             pxd.portworx.com                Delete          Immediate           true                   12d
px-csi-db-cloud-snapshot-encrypted   pxd.portworx.com                Delete          Immediate           true                   12d
px-csi-db-encrypted                  pxd.portworx.com                Delete          Immediate           true                   12d
px-csi-db-local-snapshot             pxd.portworx.com                Delete          Immediate           true                   12d
px-csi-db-local-snapshot-encrypted   pxd.portworx.com                Delete          Immediate           true                   12d
px-csi-replicated                    pxd.portworx.com                Delete          Immediate           true                   12d
px-csi-replicated-encrypted          pxd.portworx.com                Delete          Immediate           true                   12d
```