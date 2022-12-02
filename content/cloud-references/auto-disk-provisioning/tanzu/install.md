---
title: Install and configure Portworx on VMware Tanzu
description: Learn to scale a Portworx cluster up or down on VMware Tanzu with Auto Scaling.
keywords: Automatic Disk Provisioning, Dynamic Disk Provisioning, VMWare, tanzu, vSphere ASG, Kubernetes, k8s
linkTitle: Install
weight: 300
noicon: true
---

{{<info>}}
**NOTE:** 

* Portworx does not support Tanzu Community Edition
* The installation steps below only apply if you're running with Kubernetes on the TKG or TKGS platforms
* For the underlying datastore for the PVs and VMs, **do not enable Storage DRS**. VSphere CSI driver and Cloud Native Storage [does not currently support Storage DRS feature in vSphere](https://vsphere-csi-driver.sigs.k8s.io/supported_features_matrix.html)
{{</info>}}

## Prerequisites

* A running Kubernetes cluster TKG or TKGS.
* If you're using TKGS, ensure all prerequisites are met based on [VMware documentation](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-3040E41B-8A54-4D23-8796-A123E7CAE3BA.html#prerequisites-1).
* Enable Workload Management according to [VMware Best Practices](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-3040E41B-8A54-4D23-8796-A123E7CAE3BA.html).
* Ensure the required ports in the `9001:9020`  range are open on all nodes.

{{<info>}}
**NOTE**:
If you're using encrypted volumes on Tanzu clusters with PhotonOS, Portworx recommends that you install the following packages all nodes:

- device-mapper
- cryptsetup

You can install these packages using the following commands:

```text
yum update -y
yum install device-mapper
yum install cryptsetup
```
{{</info>}}

## Create the StorageClass

CSI cloud drive configuration requires a StorageClass with the CSI driver set as a provisioner to be installed in the Kubernetes cluster.

1. Get a CSI provisioner name by running the following command on your Tanzu Kubernetes cluster. You will use this name in the next step:
  
    ```text
    kubectl get csidriver
    ```

2. Create a StorageClass on your specified datastore. In the `provisioner` field, enter the CSI driver name you got in the step above. Provide the datastore URL in the `datastoreurl` field from your VSpere client. 

    ```text 
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: vsphere-immediate-sc
      annotations:
         storageclass.kubernetes.io/is-default-class: "true"
    provisioner: <csi_driver_name>
    parameters:
      datastoreurl: "<your-datastore-url>" 
    allowVolumeExpansion: true
    reclaimPolicy: Delete
    volumeBindingMode: Immediate
    ```
    {{<info>}}**NOTE:** You can also specify the SPBM policies in the StorageClass with the `storagepolicyname` parameter.{{</info>}}

## Install Portworx

As part of the installation, you'll use the spec generator. Perform the following steps to create your Portworx spec and configure your Cloud Drives. This installation method uses the Operator. 

To install Portworx with Kubernetes, you must first generate Kubernetes manifests that you will deploy in your cluster:

 1. Navigate to <a href="https://central.portworx.com" target="tab">PX-Central</a> and log in, or create an account
   
 2. Click **Continue** with Portworx Enterprise option:

    ![Screenshot showing install and run](/img/pxcentral-install.png)

 3. Choose either **{{< pxEnterprise >}}** or **{{< pxEssentials >}}**, depending on which license you intend to use:

    ![Screenshot showing Portworx license selector](/img/pxcentral-license.png)

 5. Select VMware Tanzu Cloud Platform and specify disk size and storage class name you will be using for drives

    ![Screenshot showing Tanzu configuration](/img/wmvare-tanzu-configuration.png)

6. On the **Customize** tab, add the following under the Environment Variables section:

    * In the `name` field, add `PRE-EXEC`
    * In the `value` field, add `iptables -A INPUT -p tcp --match multiport --dports 9001:9020 -j ACCEPT`
