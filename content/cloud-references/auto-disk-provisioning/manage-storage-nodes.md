---
title: Manage the number of storage nodes in a cluster
linkTitle: Manage storage nodes
keywords: storageless nodes, max storage nodes per zone, cloud drives, maxStorageNodesPerZone. limit storage nodes
description: Understand how the Portworx manages the number of storage nodes in a cluster.
---

This page explains how Portworx automatically manages the number of storage nodes in a cluster. It also explains the need for this behavior and how you can override it.


## Why limit the number of storage nodes?

The value of the `spec.cloudStorage.maxStorageNodesPerZone` parameter gates the number of storage nodes that can exist in a single failure domain. The following example describes an issue that you can encounter when this parameter is not set.

Consider a cluster with a single storage node per zone where `maxStorageNodesPerZone` is not set:

![A cluster with a single storage node per zone](/img/max-storage-nodes-img1.png)


If your cloud provider decides to recycle the _N1_ node, then _N1-new_ is the new node that replaces the node _N1_. It is often observed that the _N1-new_ node starts before the _N1_ node is terminated, and that the _N1_ node may not have relinquished its disk. Thus, Portworx cannot reuse those disks in the new node _N1-new_, which results in creation of a new storage node with a new cloud drive, as shown in the following diagram:

![New node creates new cloud drive](/img/max-storage-node-img2.png)

In this case, if the value of `maxStorageNodesPerZone` had seen set to 1, then _N1-new_ node would not have created the new cloud drive _Px disk-4_.

## How does Portworx manage storage nodes?


Portworx automatically manages the number of storage nodes in a new cluster[^1]. This is achieved by choosing a value for the `maxStorageNodesPerZone` parameter[^2] as follows and avoids the issue explained above:

|Architecture | maxStorageNodesPerZone|
|---|---|
|[Disaggregated install](/install-portworx/disaggregated/)|_(Total nodes marked as `storage`) / (Total zones)[^3]_|
|[Converged install](/cloud-references/deployment-arch/)|_(Total nodes)[^4] / (Total zones)[^3]_|


Here are some examples of how Portworx chooses the default value of `maxStorageNodesPerZone` using the above formulas:

|Number of zones|Total nodes or Total Nodes marked storage|Default value of `maxStorageNodesPerZone`|
|----|----------|------------|
|1|10|10|
|3|9|3|
|3|10|3|


## How do I override the value of maxStorageNodesPerZone?

Depending on whether you are performing a fresh install or modifying an existing installation, proceed to one of the following sections.

### A fresh installation

You can use any one of the following methods to set a value for `maxStorageNodesPerZone` so that Portworx does not set this value automatically:

* While generating a Portworx spec from [PX-Central](https://central.portworx.com/specGen/wizard), perform the following steps:
    1. On the **Basic** page, select the latest Portworx version from the **Portworx Version** dropdown and click **Next**. 
    2. On the **Storage** page, select a cloud provider and then specify a number in the **Max storage nodes per availability zone** option.

    
* Add the following to your generated Portworx spec:

    ```text 
    spec:
      cloudStorage:
        maxStorageNodesPerZone: {{<highlight>}}<custom-value>{{</highlight>}}
    ```
{{<info>}}**NOTE:** 0 is a special value for `maxStorageNodesPerZone`, which keeps the number of storage nodes unbounded in a cluster. Every node that joins the cluster will be a storage node. This value must be used with great caution to avoid the situation mentioned [previously](#why-limit-the-number-of-storage-nodes).
{{</info>}}

### An existing installation

Add more storage nodes by increasing the value of `maxStorageNodesPerZone` in the StorageCluster spec.

{{<info>}}**NOTE:** If you are using Portworx version 2.12 or earlier, you must decommission the storageless nodes, and the new nodes will come up as storage nodes. Starting with version 2.12.1, the storageless nodes are automatically converted into storage nodes.
{{</info>}}

## How is a zone determined?

For different environments, the following table shows how zones are determined:

| Cloud provider| Zone|
|----------|--------------------|
|Public clouds, such as AWS, Azure, Google Cloud, or IBM Cloud| Detected by Portworx using cloud providers APIs.|
| VMware vSphere or VMware Tanzu | <ul><li>The value of the zone label `topology.portworx.io/zone`[^5] determines the zone for the node.</li><li> Single zone. If the zone label is not specified, every node is considered to be in a single `default` zone.</li></ul>|
|Pure FlashArray| Single zone. No zoning labels are supported.|



[^1]: When an existing cluster upgrades to any newer version of Portworx, `maxStorageNodesPerZone` is updated using the same formula mentioned above. Portworx can perform this update only if `maxStorageNodesPerZone` is not specified for the existing cluster.
[^2]: If the user account has permissions to access Auto Scaling Groups (ASG) information, Portworx honors the ASG settings. If the minimum value in ASG is lower than `maxStorageNodesPerZone`, Portworx creates storage nodes equal to the minimum value.
[^3]: If the number of nodes is unevenly distributed, the zone with the smallest number of nodes is chosen. For example, if there are 10 nodes across 3 zones, the value 3 is selected. 
[^4]: For the total number of nodes, Kubernetes master and control plane nodes are excluded if they are part of the cluster.
[^5]: For Portworx versions 2.12.0 or earlier, the supported label is `failure-domain.beta.kubernetes.io/zone`.
