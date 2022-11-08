---
title: Install Portworx on PKS using the Operator
linkTitle: Install using the Operator
logo: /logos/pks.png
weight: 200
description: Install, on-premises PKS, Pivotal Container Service, kubernetes, k8s, air gapped
keywords: portworx, PKS, kubernetes
noicon: true
aliases:
    - /portworx-install-with-kubernetes/on-premise/install-pks/operator/
    - /portworx-install-with-kubernetes/on-premise/install-pks/daemonset/
    - /install-portworx/on-premises/install-pks/daemonset/
    - /portworx-install-with-kubernetes/cloud/pks
    - /install-portworx/cloud/install-pks/
    - /portworx-install-with-kubernetes/on-premise/install-pks/install-cfcr-etcd-release/
    - /install-portworx/on-premises/install-pks/install-cfcr-etcd-release/
---

{{< content "shared/on-prem-pks-common-install.md" >}}

### Architecture

{{< content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-shared-arch.md" >}}

### Install the Operator

Enter the following `kubectl create` command to deploy the Operator:

```text
kubectl create -f https://install.portworx.com/?comp=pxoperator
```

### ESXi datastore preparation

Create one or more shared datastore(s) or datastore cluster(s) which is dedicated for Portworx storage. Use a common prefix for the names of the datastores or datastore cluster(s). We will be giving this prefix during Portworx installation later in this guide.

<!-- Steps 1 and 2 should be common with the vSphere auto-provisioning section 
There is a shared section already here "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-install-common.md", but Step 3 is different for PKS and common vSphere deployment
I talked to Nathan to split that document, but it is better docs team does that.
-->

### Step 1: vCenter user for Portworx

You will need to provide Portworx with a vCenter server user that will need to either have the full Admin role or, for increased security, a custom-created role with the following minimum [vSphere privileges](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.security.doc/GUID-FEAB5DF5-F7A2-412D-BF3D-7420A355AE8F.html):

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

If you create a custom role as above, make sure to select "Propagate to children" when assigning the user to the role.

{{<info>}}All commands in the subsequent steps need to be run on a machine with kubectl access.{{</info>}}

### Step 2: Create a Kubernetes secret with your vCenter user and password

{{< content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-secret.md" >}}

### Step 3: Generate the specs

Generate the spec file using the Portworx [Spec Generator](https://central.portworx.com/specGen/wizard) with the following configurations:

1. Under the **Basic** tab, ensure that the following config parameters are set.
    - Select the **Use the Portworx operator** checkbox.
    - Select the **{{% currentVersion %}}** version of Portworx in **Portworx Version** drop-down. 
    - Under **ETCD**, select **Built-in**.

2. Under the **Storage** tab, ensure that the following config parameters are set.
    - Under **Select your environment**, choose **Cloud**.
    - Under **Select Cloud platform**, select **vSphere**.
    - Under **Configure storage devices**, choose **Create Using a Spec** in **Select type of disk**.
    - Under **vCenter Endpoint**, type your vCenter IP or hostname.
    - Under **vCenter datastore prefix**, type the prefix of the datastore for Portworx to use.

3. Under the **Customize** tab, ensure that the following config parameters are set.
    - In **Are you running either of these?**, choose **PKS (Pivotal Container Service)**.
    - In **Advanced settings**, select the following checkboxes. 
        - **Enable Stork**
        - **Enable CSI**
        - **Enable Monitoring**
        - **Enable Telemetry**

<!-- This section below was part of a shared section title called "vsphere-pks-generate-spec-internal-kvdb.md" but this has changed and will no longer be shared. -->

{{< content "shared/operator-apply-the-spec.md" >}}

{{< content "shared/operator-monitor.md" >}}