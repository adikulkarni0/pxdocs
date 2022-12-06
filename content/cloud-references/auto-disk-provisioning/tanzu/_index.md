---
title: Disk Provisioning on VMware Tanzu
description: Learn to scale a Portworx cluster up or down on VMware Tanzu with Auto Scaling.
keywords: Automatic Disk Provisioning, Dynamic Disk Provisioning, VMWare, tanzu, vSphere ASG, Kubernetes, k8s
linkTitle: VMware Tanzu
weight: 300
series: "auto-disk-provisioning"
---

This guide explains how the Portworx Dynamic Disk Provisioning feature works within Kubernetes on VMware Tanzu.

{{<info>}}
**NOTE**: Portworx does not support Tanzu Community Edition.
{{</info>}}


## Architecture

When Portworx is deployed within Tanzu, the Cloud Drive (CD) module does the following:

* Initializes a Portworx node which involves joining the Tanzu cluster as a storage capable node, adding to the global capacity of the cluster.
* The CD module executes a subsequence of calls to check the following points:
    
    * If there are a set of drives attached to the node, the CD module uses them. If they are not already attached to the node, the CD module attaches them.
    * If there are no drives attached or available for that node, the CD module creates and attaches them.

* The CSI implementation in cloud drives makes calls to Kubernetes instead of talking to the underlying storage API. 

## CSI drive management

During installation, Portworx users provide a drive spec that defines the desired drive size. The internal CSI provider (a part of Cloud Drives that implements the CSI) parses the spec and prepares objects for creation.

The following diagram demonstrates how Portworx operates the cloud drives:

<!-- <div style="width: 80%;"> -->
![Screenshot showing Tanzu configuration](/img/vmWareTanzu.png)
<!-- </div> -->

To create and attach a drive, Portworx creates a Persistent Volume Claim (PVC). The PVC requires a StorageClass with a provisioner set to the storage provider driver name. If a StorageClass is not provided as a parameter, Portworx will use the cluster default StorageClass.  

Portworx uses the Persistent Volume (PV) ID to create a VolumeAttachment CRD. The CSI driver attaches drives through the component called external attacher. It runs as a sidecar and watches for newly created VolumeAttachment and attaches a physical drive to a node.

## Drive capacity management

The CSI driver has another component, the external-resizer. This is also deployed as a sidecar container. It implements the logic of watching the Kubernetes API for Persistent Volume claim edits, issuing the ControllerExpandVolume RPC call against a CSI endpoint, and updating the PersistentVolume object to reflect the new size.

When the actual storage usage grows, the cluster capacity can be increased, and the Cloud Drives will modify a PVC to expand its requested storage size. The external-resizer will trigger an expansion in the volume associated with the PVC in vSphere Cloud Native Storage, which finally gets reflected on the corresponding PV object's capacity.

## Node failure handling

When Portworx initializes on a node at a high level, it looks for existing cloud drives that are available to use before creating new cloud drives. The creation of a new cloud drive initializes a brand new storage node in the cluster. If a node attaches an existing cloud drive, it will re-use that cloud drive set's identity and disks. This results in the recovery of a node that was previously used to attach that cloud drive set.

The Portworx node recovery mechanism is essential in the following scenarios:

* A Portworx storage node is terminated and replaced. The reason for termination can be upgrading the OS or its packages, upgrading Kubernetes, updating the node's specs.
In this case, when the new node comes up, it will look up the available cloud drive set and will find the drive set of terminated storage node as available (because it's no longer attached). It will attach that drive set and resume the identity of the removed node.

* A Portworx storage node is terminated permanently, or Portworx running on the storage node runs into a failure and does not come up.

In this case, if a storageless Portworx node is present, it will detect that the storage node has gone down. The storageless node will find the cloud drive set of the terminated node as available and attach it to gain the identity of the terminated storage node.
