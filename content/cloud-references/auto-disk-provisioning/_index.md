---
title: Automatic disk provisioning
description: Understand how Portworx provisions disks automatically on various cloud platforms
keywords: Automatic Disk Provisioning, Dynamic Disk Provision, Cloud
weight: 200
series: arch-references
hidesections: true
---

In cloud environments, Portworx can dynamically create disks based on an input disk template whenever a new instance spins up and use those disks for the Portworx cluster.

Portworx fingerprints the disks and attaches them to an instance in the autoscaling cluster. In this way an ephemeral instance gets its own identity.

## Why would I need this?

* Users don't have to manage the lifecycle of disks. Instead, they just have to provide disks specs and Portworx manages the disk lifecycle.
* When an instance terminates, the auto scaling group will automatically add a new instance to the cluster. Portworx gracefully handles this scenario by re-attaching the disks to it and give a new instance the old identity. This ensures that the instance’s data is retained with zero storage downtime.

### Limit the number of storage nodes 

For the [two deployment architectures](/cloud-references/deployment-arch/), you can manage the number of storage nodes as follows:

|Architecture| Strategy|
|------------|------------|
|Converged|`spec.cloudStorage.maxStorageNodesPerZone` setting is used to control the number of storage nodes in the cluster.|
|Disaggregated|Only the nodes labeled as `portworx.io/node-type=storage` are converted to storage nodes.<br><br>Note: If `maxStorageNodesPerZone` is specified and its value is less than the number of nodes labeled as storage, then `maxStorageNodesPerZone` takes precedence.


Portworx automatically manages the number of storage nodes in a cluster. For more information about why automatic management is necessary and how it is implemented, see [Manage the number of storage nodes](/cloud-references/auto-disk-provisioning/manage-storage-nodes).

{{<info>}}**NOTE:** If a cluster has no zones, all nodes are assumed to be in a single zone. For example, most vSphere setups and all FlashArray cloud setups are considered to be in a single zone.{{</info>}}

## How do I set it up?

On Kubernetes, when generating the spec using the Portworx spec generator in [PX-Central](https://central.portworx.com), the UI will prompt for the disk specs in the cloud section of the wizard.

Based on the cloud platform, you will need to provide access to Portworx to the cloud APIs. Continue to below sections for additional details.

{{<homelist series="auto-disk-provisioning">}}