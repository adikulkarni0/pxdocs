---
title: "Prerequisites"
weight: 100
disableprevnext: true
equalizeTableWidths: "50%"
description: Start your installation.
keywords: Portworx, containers, storage
aliases:
    - /start-here-installation/
---

## Installation Prerequisites

The minimum supported size for a Portworx cluster is three nodes. Each node must meet the following hardware, software, and network requirements:

|**Hardware** ||
|-------------------------|------------|
|     CPU | 4 cores minimum, 8 cores recommended |
|     RAM | 4GB minimum, 8GB recommended |
| Disk <ul><li>`/var`</li><li>`/opt`</li></ul> | <ul><li>2GB free</li><li>3GB free</li></ul> |
|Backing drive | 8GB (minimum required)<br/>128 GB (minimum recommended)|
|Operating system root partition | 64 GB is the minimum required size for the root filesystem which contains the operating system<br>128 GB minimum recommended|
|Storage drives | Storage drives must be unmounted block storage: raw disks, drive partitions, LVM, or cloud block storage. |
|Network connectivity | Bandwidth: <ul><li>10 Gbps recommended</li><li>1 Gbps minimum</li></ul><br/>Latency requirements for synchronous replication: less than 10ms between nodes in the cluster|

|**Network** ||
|--- | ---|
|Open needed ports | Portworx requires different open ports depending on how it's installed:<ul><li>Spec-based installations require all Portworx nodes to have open TCP ports at 9001-9022 and an open UDP port at 9002.</li><li>Portworx on OpenShift 4+ requires open TCP ports at 17001-17020 and an open UDP port at 17002.</li><li>Telemetry components require the following open ports: 9024, 12001 and 12002.</li></ul>Portworx also requires an open KVDB port. For example, if you're using `etcd` externally, open port 2379.<br/><br/>If you intend to use Portworx with sharedv4 volumes, you may need to [open your NFS ports](/operations/operate-kubernetes/storage-operations/create-pvcs/open-nfs-ports).|

|**Software** ||
|--- | ---|
|Linux kernel | Version 3.10 or greater.|
|Docker | Version 1.13.1 or greater.|
|Key-value store | Portworx needs a key-value store to perform its operations. As such, install a clustered key-value database \(`etcd`\) with a three node cluster.<br><br>With Portworx 2.0 or greater, you can use Internal KVDB during installation. In this mode, Portworx will create and manage an internal key-value store (kvdb) cluster.<br><br>If you plan of using your own etcd, refer to [Etcd for Portworx](/operations/etcd) for details on recommendations for installing and tuning etcd.|
|Disable swap|Please disable swap on all nodes that will run the Portworx software.  Ensure that the swap device is not automatically mounted on server reboot.|

## Supported Kubernetes versions

Before you install Portworx on Kubernetes, ensure that you're using a supported Kubernetes version:

{{<automateTable source="supportedK8s">}}

{{<info>}}
**K3s users:** You must use CSI integration to generate / use PVCs.
{{</info>}}

## Air-gapped prerequisites

If you intend to use the `sharedv4` feature, your host systems must be running the NFS service. For more information on options for installing the NFS service, refer to the installation article for air-gapped clusters. 

## Installation

Whether you're using {{< pxEnterprise >}} or Essentials, you can install Portworx on the cloud or on-premises. Proceed to one of the following sections for Kubernetes and OpenShift install instructions.

{{<homelist series="k8s-install">}}

For all other environments, continue to the following section:

{{< widelink url="/install-with-other" >}}Portworx on other orchestrators{{</widelink>}}

## Post-installation

If you have an existing Portworx cluster, continue to below sections for using and managing Portworx.

{{<homelist series2="k8s-postinstall">}}