---
title: Install Portworx on vSphere with Amazon EKS Anywhere
linkTitle: Amazon EKS Anywhere
description: This section describes lists the steps to deploy Portworx on vSphere having Amazon EKS Anywhere.
keywords: portworx, VMware, vSphere, EKS Anywhere, Amazon EKS Anywhere
aliases: 
    - /portworx-install-with-kubernetes/on-premise/aws/eks-anywhere/portworx-vsphere-eks-anywhere/
---


Portworx can be installed on a Kubernetes cluster running on vSphere and managed by [Amazon EKS Anywhere](https://anywhere.eks.amazonaws.com).

### Prerequisites

Before you install Portworx on vSphere, ensure that you meet the following prerequisites:

|**Environment**|**Resources**|
|------|------|
|Install<br><br>**Note:** Default Bottlerocket base image is not supported.| Ubuntu 20.04.4 LTS <br> Ubuntu 18.04.6 LTS|
|Deployment Host<br><br>**Note**: The same vSphere host where you deploy EKS-Anywhere.| VM OS: Linux <br/>vCPU: 4 <br/> Memory: 16 GB <br/> Disk storage: 200 GB|
|Control-Plane VMs| Minimum: 1 <br/> Recommended: 3 <br/>vCPUs: 2 <br/> RAM: 8 GB <br/> OS Volume: 25 GB|
|Worker Node VMs| Minimum: 3 (for storage cluster) <br/> vCPUs: 8 <br/>RAM: 16 GB <br/> OS Volume: 25 GB|

### Step 1: vCenter user for Portworx

Provide Portworx with a vCenter server user that has either the full admin role, or for increased security, a custom-created role with the following minimum [vSphere privileges](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.security.doc/GUID-FEAB5DF5-F7A2-412D-BF3D-7420A355AE8F.html):

* Datastore
    * Allocate space
    * Browse datastore
    * Low level file operations
    * Remove file
* Host
    * Local operations
    * Reconfigure virtual machine
* Virtual machine
    * Change Configuration
    * Add existing disk
    * Add new disk
    * Add or remove device
    * Advanced configuration
    * Change Settings
    * Extend virtual disk
    * Modify device settings
    * Remove disk

If you created a custom role with the permissions above, select **Propagate to children** when assigning the user to the role.

{{<info>}}
**NOTE:** All commands in the subsequent steps need to be run on a machine with `kubectl` access.
{{</info>}}

### Step 2: Create a Kubernetes secret with your vCenter user and password

{{< content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-secret.md" >}}

### Step 3: Generate Portworx spec

1. Navigate to [PX-Central](https://central.portworx.com/) and log in, or create an account.
2. Select **Portworx Enterprise** and click **Continue**.
3. On the **Basic** page, ensure that **Use the Portworx Operator** is selected. Select the latest version of Portworx from the **Portworx Version** drop-down, the **Built-in** option for etcd, and then click **Next**.

2. On the **Storage** page, select **Cloud** as your environment, and **vSphere** as your cloud platform. Select the **Create Using a Spec** option to configure your storage devices, specify the following, and click **Next**:

    * In the **vCenter Endpoint** field, specify the hostname or IP address of your vCenter server.
    * In the **vCenter datastore prefix** field, specify the prefix name of the vCenter datastore you want to use.
    * In the **Kubernetes Secret Name** field, specify the name of the secret that you specified in [Step 2](#step-2-create-a-kubernetes-secret-with-your-vcenter-user-and-password), which is `px-vsphere-secret`.
    * In the **vCenter Port** field, enter port 443, if it is not auto  filled.

3. Choose your network and click **Next**.

4. On the **Customize** page, enable Stork, CSI, Monitoring, and Telemetry in **Advanced Settings** and click **Finish** to generate the spec.

{{< content "shared/portworx-install-with-kubernetes-4-apply-the-spec.md" >}}

