---
title: Operate IBM Cloud Drives
logo: /logos/ibm.png
linkTitle: Operate and troubleshoot IBM Cloud Drives
weight: 500
keywords: IBM Cloud Kubernetes Service, IBM cloud drive, VPC Gen-2
description: Learn how to operate and troubleshoot Portworx on IBM VPC Gen2 Cloud with cloud drives.
noicon: true
aliases:
    - /portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/cloud-drive-operations/IBM/operate-cloud-drives/
---
This page describes operations and troubleshooting for Portworx clusters on IBM VPC Gen2 Cloud with cloud drives

## Operate

### Limit storage nodes

To control the number of storage nodes in the cluster, use the `spec.cloudStorage.maxStorageNodesPerZone` or `spec.cloudStorage.maxStorageNodes` configuration in your [StorageCluster](/reference/crd/storage-cluster/).
The total number of storage nodes in your cluster will be the number of zones multiplied by `max_storage_nodes_per_zone`.

Example:

```text
"-s", "sc=ibmc-vpc-block-10iops-tier,size=200", "-max_storage_nodes_per_zone", "1"
```

For a cluster of 6 nodes spanning 3 zones (us-south-1, us-south-2, and us-south-3), in the above example Portworx will have 3 storage nodes (one in each zone) and 3 storageless nodes. Portworx will create a total 3 data disks, each of size 200, and will attach one disk to each storage node.


### Scale down clusters

A Portworx cluster on IBM VPC Gen2 with cloud drives can contain a mixture of storage and storageless nodes.
When scaling down an IBM IKS/ROKS cluster, do not scale down the cluster size lower than the total number of storage nodes in the Portworx cluster.

You can find out the number of storage nodes by running `pxctl status` on any Portworx node. This lists all nodes in the cluster and specifies which ones are storage nodes.

### Node failure handling

When Portworx initializes on a node at a high level, it looks for existing cloud drives that are available to use before creating new cloud drives. The creation of a new cloud drive initializes a brand new storage node in the cluster. If a node attaches an existing cloud drive, it will re-use that cloud drive set’s identity and disks. This results in the recovery of the node that was previously used to attach that cloud drive set.

The Portworx node recovery mechanism is essential in the following scenarios:

* A Portworx storage node is terminated and replaced. The reason for termination could be upgrading the OS or its packages, upgrading Kubernetes, or updating the node’s specs. In this case, when the new node comes up, it looks up the available cloud drive set and finds the drive set of the terminated storage node as available (because it’s no longer attached). It then attaches that drive set and resumes the identity of the removed node.
* A Portworx storage node is terminated permanently, or Portworx running on the storage node runs into a failure and does not come up.

In this case, if a storageless Portworx node is present, it detects that the storage node has gone down. The storageless node finds the cloud drive set of the terminated node as available and attaches it to gain the identity of the terminated storage node.


### Portworx storage pool expansion

See [Expand your storage pool size](/operations/operate-kubernetes/storage-operations/create-pvcs/expand-storage-pool/).


## Troubleshoot

### Recover a node with a deleted VolumeAttachment

If a VolumeAttachment gets deleted, the node will not boot and will return an error such as `cannot delete va, name: <va-name>`.

To recover from this state, perform a node replace operation from the IBM Cloud UI.

## Related topics

* [Install and configure OpenShift (OCP) on IBM Cloud](/install-portworx/cloud/ibm/ocp-on-ibm-cloud/)
