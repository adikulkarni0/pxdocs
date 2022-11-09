---
title: Configure Pure Storage FlashBlade as a Direct Access filesystem
weight: 
linkTitle: Pure FlashBlade PVCs
keywords: 
description: 
aliases:
    - /portworx-install-with-kubernetes/storage-operations/create-pvcs/pure-flashblade/
---
On-premises users who want to use Pure Storage FlashBlade with Portworx on Kubernetes can attach FlashBlade as a Direct Access filesystem. Used in this way, Portworx directly provisions FlashBlade NFS filesystems, maps them to a user PVC, and mounts them to pods. Once mounted, Portworx writes data directly onto FlashBlade. As a result, this mounting method doesn't use storage pools.

FlashBlade Direct Access filesystems support the following:

* Basic filesystem operations: create, mount, expand, unmount, delete
* NFS export rules: Control which nodes can access an NFS filesystem 
* Mount options: Configure connection and protocol information
* NFS v3 and v4.1

{{<info>}}
**NOTE:** 

* FlashBlade Direct Access filesystems do not support subpaths. 
* Autopilot does not support FlashBlade volumes.
{{</info>}}

### Mount options

You specify mount options through the CSI `mountOptions` flag in the `storageClass` spec. If you do not specify any options, Portworx will use the default options from the client side command instead of its own default options.

Mount options rely on the underlying host operating system and Purity//FB version. Refer to the FlashBlade [documentation](https://support.purestorage.com/FlashBlade/Purity_FB/FlashBlade_User_Guides) for more information on specific mount options available to you.

### NFS export rules

NFS export rules define access writes and privileges for a filesystem exported from FlashBlade to an NFS client. 

### Differences between FlashBlade Direct Access filesystems and proxy volumes

Direct Access dynamically creates filesystems on FlashBlade that are managed by Portworx on demand, while proxy volumes are created by users and then used by Portworx as required.

The following existing Portworx parameters don't apply to Pure Direct Access filesystems:

  * shared
  * sharedv4
  * secure
  * repl
  * scale should be 0
  * aggregation_level should be less than 2

### Direct Access Architecture

Portworx runs on each node. When a user creates a PVC, Portworx provisions an NFS filesystem on FlashBlade and maps it directly to that PVC based on configuration information provided in the `storageClass` spec.

![](/img/FBarch.png)
## Install Portworx and configure FlashBlade

Before you install Portworx, ensure that your physical network is configured appropriately and that you meet the prerequisites. You must provide Portworx with your FlashBlade configuration details during installation. 

### Prerequisites

* FlashBlade must be running Purity//FB version 2.2.0 or greater. Refer to the [Supported models and versions](/reference/pure-reference/supported-versions/) topic for more information. 
* Your cluster must have local or cloud drives accessible on each node. Portworx needs local or cloud drives on the node (block devices) for the journal and for at least one storage pool. 
* The latest NFS software package installed on your operating system (nfs-utils or nfs-common)
* FlashBade can be accessed as a shared resource from all the cluster nodes. Specifically, both `NFSEndPoint` and `MgmtEndPoint` IP addresses must be accessible from all nodes. 
* You've set up the secret, management endpoint, and API token on your FlashBlade.
* If you want to use Stork as the main scheduler, you must use Stork version 2.12.0 or greater.

### Deploy Portworx

Once you've ensured you meet the prerequisites and your physical network topology is appropriately configured, you're ready to deploy Portworx.

1. Create a JSON file named `pure.json` that contains your FlashBlade information:

    ```text
    {
      "FlashBlades": [
        {
      	  "MgmtEndPoint": "<fb-management-endpoint>",
      	  "APIToken": "<fb-api-token>",
      	  "NFSEndPoint": "<fb-nfs-endpoint>"
      	}
      ]
    }
    ```

    {{<info>}}
**NOTE:** You can add FlashArray configuration information to this file if you're configuring both FlashBlade and FlashArray together. Refer to the [JSON file](/reference/pure-reference/pure-json-reference/) reference for more information.
    {{</info>}}

2. Enter the following `kubectl create` command to create a Kubernetes secret called `px-pure-secret`:
    
    ```text
    kubectl create secret generic px-pure-secret --namespace kube-system --from-file=pure.json=<file path>
    ```
    ```output
    secret/px-pure-secret created
    ```
   
    {{<info>}}
**NOTE:** You must name the secret `px-pure-secret`.
    {{</info>}}


3. Deploy Portworx [on your on-premises Kubernetes cluster](/install-portworx/on-premises/other/). Ensure **CSI** is enabled.

Once deployed, Portworx detects that the FlashBlade secret is present when it starts up, and can use the specified FlashBlade as a Direct Access filesystem.

## Use FlashBlade as a Direct Access filesystem

Once you've configured Portworx to work with your FlashBlade, you can create a StorageClass and reference it in any PVCs you create. 
### Create a StorageClass

Create a StorageClass spec, specifying your own values for the following:

* **parameters.backend:** with `pure_file` 
* **parameters.pure_export_rules** with any NFS [export rules](https://support.purestorage.com/FlashBlade/Purity_FB/Data_Protocols/NFSv3/Mounting_a_File_System_Using_NFS_to_a_Client) you desire
* **mountOptions** with any CSI mount options you desire

    ```text
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: portworx-pso-fb-v3
    provisioner: pxd.portworx.com
    parameters:
      backend: "pure_file"
      pure_export_rules: "*(rw)"
    mountOptions:
      - nfsvers=3
      - tcp
    allowVolumeExpansion: true
    ```

### Create a PVC

Reference the StorageClass you created by entering the StorageClass name in the `spec.storageClassName` field:

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pure-claim-file-v3
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: portworx-pso-fb-v3
```

## Related topics

* [Pure.json file reference](/reference/pure-reference/pure-json-reference/)
* [Disk provisioning on Pure Storage FlashArray](/cloud-references/auto-disk-provisioning/pure-flash-array/)
* [CSI topology for FlashArray and FlashBlade Direct Access volumes](/operations/operate-kubernetes/cluster-topology/csi-topology)
