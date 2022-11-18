---
title: Disk Provisioning on VMware vSphere
description: Learn to scale a Portworx cluster up or down on VMware vSphere with Auto Scaling.
keywords: Automatic Disk Provisioning, Dynamic Disk Provisioning, VMWare, vSphere ASG, Kubernetes, k8s
linkTitle: VMware vSphere
weight: 300
noicon: true
---

This guide explains how the Portworx Dynamic Disk Provisioning feature works within Kubernetes on VMware and the requirements for it.

{{<info>}}Installation steps below are only supported if you are running with Kubernetes.{{</info>}}

## Architecture

{{< content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-shared-arch.md" >}}


## Limiting storage nodes

{{< content "shared/cloud-references-auto-disk-provisioning-asg-limit-storage-nodes.md" >}}

{{< content "shared/cloud-references-auto-disk-provisioning-asg-examples-vsphere.md" >}}

## Availability across failure domains

Since Portworx is a storage overlay that automatically replicates your data, {{<companyName>}} recommends using multiple availability zones when creating your VMware vSphere based cluster. Portworx automatically detects regions and zones that are populated using known Kubernetes node labels. You can also label nodes with custom labels to inform Portworx about region, zones and racks. Refer to the [Cluster Topology awareness](/operations/operate-kubernetes/cluster-topology/) page for more details.

## Install Portworx

Follow the instructions in this section to install Portworx.

### Prerequisites 

* VMware vSphere version 7.0 or newer.
* `kubectl` configured on the machine having access to your cluster.
* Portworx does not support the movement of VMDK files from the datastores on which they were created. Do not move them manually or have any settings that would result in a movement of these files. In order to make sure that Storage DRS does not move VMDK files, set Storage DRS settings to the following configuration.

    {{<info>}}**NOTE:** If you provide a vSphere datastore cluster as an input, Portworx uses a DRS recommended API to select an appropriate datastore from the list of datastores in the datastore cluster.{{</info>}}


### Enable Storage DRS configuration

 From the **Edit Storage DRS Settings** window of your selected datastore cluster, edit the following settings:

*  For **Storage DRS automation**, choose the **No Automation (Manual Mode)** option, and set the same for other settings, as shown in the following screencapture: 

    ![VsphereDRS1](/img/VsphereDRS1.png)

* For **Runtime Settings**, clear the **Enable I/O metric for SDRS recommendations** option.    

    ![VsphereDRS1](/img/VsphereDRS2.png)

* For **Advanced options**, clear the **Keep VMDKs together by default** options.

    ![VsphereDRS1](/img/VsphereDRS3.png)

{{< content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-px-install.md" >}}
