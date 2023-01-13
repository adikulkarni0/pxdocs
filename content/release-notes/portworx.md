---
title: Portworx Release Notes
weight: 20000
description: Notes on Portworx releases.
keywords: portworx, release notes
series: release-notes
aliases:
    - /reference/release-notes/portworx
---

## 2.12.1

December 14, 2022


### New features

Portworx by Pure Storage is proud to introduce the following new features:

* Google Cloud users can now [encrypt GCP cloud drives using customer managed encryption keys](/operations/operate-kubernetes/cloud-drive-operations/gcp/).

* You can now use [Vault Transit](/operations/key-management/vault-transit) to manage key generation for encrypting in-transit data.


### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
| ---- | ---- |
| PWX-26232	| The  Portworx node's IP addresses are now included in the license server's "long client usage" output (`lsctl client ls -l`). |
| PWX-26304	| Storageless nodes will become storage nodes when `max_storage_nodes_per_zone` is increased. |
| PWX-27769	| vSphere and IBM cloud platforms will be able to now recognize the zone label `topology.portworx.io/zone`. This will help Portworx honor zone related settings like `maxStorageNodesPerZone`. |
| PWX-27174 | `pxctl cluster provision-status` will now show IP addresses for nodes. The IP addresses help you to correlate that command's output with the node list provided by `pxctl status`. |

### Fixes

| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PWX-27748	| Some incremental fixes in version 2.12.0 introduced issues with DaemonSet YAML generation for K3s and RKE2 Kubernetes platforms.<br><br>**Resolution:** These issues have been fixed. |
| PWX-27849 | Kubernetes versions 1.25 and later [do no support PodSecurityPolicy] (https://kubernetes.io/docs/concepts/security/pod-security-policy/).<br><br>**Resolution:**[PX-Central](https://central.portworx.com/landing/login) does not include PodSecurityPolicy in the YAML install specs when the Kubernetes version is 1.25 or later. |
| PWX-27267 | Cloudsnaps for aggregated volumes were failing certain checks when part of the aggregated volume did not have any differential data to upload.<br><br>**Resolution:** This version fixes those checks and prevents failure due to empty differential data. |
| PWX-27246 | During a new installation of Portworx on a vSphere environment (installation in local mode), several VMs appeared as storage nodes on a single ESXi host. This was because of a race condition in the vSphere environment, which increased the number of nodes forming the cluster and affected quorum decisions.<br><br>**Resolution:** You can choose not to upgrade if you take care of the race condition during installation. You can overcome the race condition by allowing only 1 VM to come up on an ESXi host during installation. Once the installation is complete, you can bring up as many VMs as you want simultaneously. The problem exists only until the first storage VM comes up. |
| PWX-26021 |	For sharedv4 apps, multiple mount/unmount requests on the same path could become stuck in `Uninterruptible Sleep (D) State`.<br><br>**Resolution:** In the case that the client tries to mount but the previous request still exists, there will be a Kubernetes event stating that the previous request is still in progress. |
| PWX-27732 |	vSphere cloud drive labels previously contained a space, which is not compatible with Kubernetes standards and caused an error in CSI.<br><br>**Resolution:**  Portworx now replaces the space character with a dash (`-`). All other special characters will be replaced by a period (`.`). |
| PWX-27227	| If a pool rebalancing was issued with the `--dry-run` option, then Portworx created unnecessary rebalance audit keys in the KVDB. As it was not possible to delete these keys, the disk size of the KVDB increased. <br><br>**Resolution :** Portworx no longer creates audit keys when a pool rebalancing is issued with the `--dry-run` option, and Portworx deletes orphaned keys that have already been created. 
| PWX-27407	| The KVDB contained inconsistent node entries because of a race condition in the auto decommission of storageless nodes. This was causing Portworx to restart.<br><br>**Resolution:** The race condition is now handled, and Portworx ensures that no inconsistent entries are left behind in the KVDB during the decommission process. |
| PWX-27917 | Portworx ignored the value of `MaxStorageNodesPerZone` if an uneven number of nodes are labeled as `portworx.io/node-type=storage`.<br><br>**Resolution:** This issue has been fixed.

### Deprecations

The following feature has been deprecated:

* Internal objectstore support


### Known issues (Errata)

| **Issue Number** | **Issue Description** |
| ---- | ---- | 
| PD-1684 | If the `sharedv4_svc_type` parameter is not specified during ReadWriteMany volume creation, Portworx defaults to a sharedv4 service volume unless Metro DR is enabled, in which case Portworx defaults to sharedv4 (non-service) volumes. You can explicitly set the `sharedv4_svc_type` parameter in the StorageClass. If it is set to an empty string, a sharedv4 (non-service) volume is created. |
| PD-1729 | On some recent Linux kernels, back to back online resize operations of Ext4 volumes can fail. This is because of a bug in the kernel which has been fixed in the latest kernel release.<br><br>**Workaround:** Upgrade to a more recent kernel version, or restart the application pod that is using the volume. This remounts the volumes and completes the resize operation. |



## 2.12.0

October 24, 2022

### Notes

{{<info>}}
Portworx 2.12.0 requires Operator 1.10.0 and Stork 2.12.0.{{</info>}}

Visit these pages to see if you're ready to upgrade to the latest version:

* [Prerequisites](/install-portworx/prerequisites/)
* [Supported kernels](/reference/supported-kernels/)

### New features

Portworx by Pure Storage is proud to introduce the following new features:

* On-prem users can now enable [PX-Fast](/operations/operate-kubernetes/storage-operations/create-pvcs/px-fastpath-pvc/) functionality that utilizes the new [PX-StoreV2 datastore](/install-portworx/on-premises/install-px-store-v2). PX-Fast enables a new accelerated IO path for volumes and is optimized for workloads requiring consistent low latencies. PX-StoreV2 is the new Portworx datastore optimized for supporting IO intensive workloads for configurations utilizing high performance NVMe class devices. 

* Early access support for [Portworx Object Service](/operations/operate-kubernetes/storage-operations/object/). This feature allows storage admins to provision object storage buckets with various backing providers using custom Kubernetes objects.

* You can now use [Vault AppRole's Role ID and Secret ID](/operations/key-management/vault/#using-approle-authentication-method) to authenticate with Vault. Portworx will auto-generate Vault tokens to store encryption secrets and cloud credentials in Vault. 

* Metro and asynchronous disaster recovery (DR) involves migrating Kubernetes resources from a source cluster to a destination cluster. To ensure that the applications can come up correctly on the destination clusters, you may need to modify resources to work as intended on your destination cluster. The [ResourceTransformation](/reference/crd/resource-transformation-usage) feature allows you to define a set of rules that modify the Kubernetes resources before they are migrated to the destination cluster.


### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
| ---- | ---- |
| PWX-26700 | A pod that was using a sharedv4 volume might have taken a few minutes to terminate when Portworx running on one node was waiting for a response from Portworx running on another node which was down. This has been fixed by deferring the remote call when the remote node is down. |
| PWX-26631 | In one of the error paths, Portworx was listing pods in all the namespaces in the Kubernetes cluster, which caused the Portworx process to consume a large amount of memory temporarily. The pod listing is now limited to a single namespace. |
| PWX-24862 | At times, users create sharedv4 volumes unintentionally by using a sharedv4 storageClass with a `ReadWriteOnce` PVC. This is because previously Portworx created a sharedv4 volume when either the storageClass had `sharedv4: true` _or_ the PVC access mode was `ReadWriteMany/ReadOnlyMany`. To avoid these unintentional sharedv4 volumes, a sharedv4 volume is now created only if the PVC access mode is `ReadWriteMany/ReadOnlyMany`, and the `sharedv4` setting in the storageClass does not matter. This may require modification to the specs of some existing apps.<br><br>If an app expects a sharedv4 volume while using a `ReadWriteOnce` PVC, some of the pods may fail to start. The PVC will have to be modified to use `ReadWriteMany` or `ReadOnlyMany` access mode. |
| PWX-23285 | Adds uniform support for `PX_HTTP_PROXY`, `PX_HTTPS_PROXY`, and `NO_PROXY` environment variables (which are equivalent to commonly used Linux `HTTP_PROXY`, `HTTPS_PROXY` and `NO_PROXY` environment vars.)<br><br>Specifying the HTTP proxy via the cluster options is now deprecated.<br><br>Also adds support for authenticated HTTP proxy, where you specify the username and password to authenticate with an HTTP proxy. For example: `PX_HTTP_PROXY=http://user:password@myproxy.acme.org`. |
| PWX-22292 | Previously, when you updated an AKS cluster principal's password and refreshed your Kubernetes secret, you would sometimes have to restart Portworx pods to propagate the changes to Portworx. Now, Portworx automatically refreshes the AKS secrets. |
| PWX-22927 | You can now use Vault AppRole authentication for vault integration. You need to provide VAULT_ADDR, VAULT_APPROLE_ROLE_ID, VAULT_APPROLE_SECRET_ID, and VAULT_AUTH_METHOD (approle) via Kubernetes Secrets or as environment variable.|
| PWX-26421 | Portworx logs now display Vault's authentication method if login is successful. |
| PWX-24437 | Added discard stats to Grafana performance dashboard. |
|PWX-18687| Portwox now instantly detects that a replication node is down as a result of socket events, preventing high IO latencies.|
| PD-1628 | Portworx now supports [FlashBlade and FlashArray SafeMode](https://blog.purestorage.com/products/protect-your-data-from-ransomware-with-safemode-snapshots/) feature |
| PD-1634| Added support for [live migration](https://docs.openshift.com/container-platform/4.8/virt/live_migration/virt-live-migration.html) of virtual machines between nodes in the OpenShift environment. This feature works only with Stork version 2.12.|

### Fixes

| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PWX-20808 | When Portworx was configured with external etcd v3 as its key-value database, there were delays and timeouts when running the `pxctl service kvdb members` command because the provided etcd endpoints to Portworx did not match with what were used internally.<br><br>**Resolution:** Portworx now consults both configured KVDB endpoints and configuration before displaying the KVDB information.|
| PWX-24649 | Occasionally, a race condition in the initial setup of Portworx was leading to an invalid topology-zone set on cloud-drives, resulting in Portworx allocating and using more cloud-drives than configured.<br><br>**Resolution:** The startup issue has been fixed.|
| PWX-26406 | Repeated CSI `NodePublish/NodeUnpublish` API calls were resulting in Portworx using more resources because these APIs did a deep Inspect on the volume.<br><br>**Resolution**: CSI `NodePublish/NodeUnpublish` now uses fewer resources because it avoids a deep Inspect and any extra API calls to the Kubernetes API server.|
| PWX-24872 | The `pxclt cloudsnap list -x 5` command was reporting an error.<br><br>**Resolution:** The issue has been fixed. |
|PWX-26935|  When a pod with sharedv4 volume is terminated, Portworx unmounts the nfs-mounted path on the local node. When the remote NFS server node was powered off, unmount was delaying the pod termination.<br><br>**Resolution:** This issue has been fixed. |
| PWX-26578 | Portworx was taking a long time to run background tasks. On some clusters, this was causing long delays due to a large backlog.<br><br>**Resolution:** This issue has been fixed. |
| PWX-23454 | Pool and node labels that are the same as volumes were ignored when applying VolumePlacementStrategies.<br><br>**Resolution:** This issue has been fixed.|
| PWX-26445 | PVC creation could failing with an unauthorized error message if the service account token being used by Portworx is expired. <br><br>**Resolution:** For Kubernetes version 1.21 and later, refresh the `ServiceAccountToken` for the Portworx service to prevent unauthorized errors (after 1 year by default, or 3 months for an EKS cluster). |
| PWX-24785| When installing Portworx on Rancher via helm charts, some permissions on `Secret` objects were missing in the Portworx spec.<br><br>**Resolution:** `px-role` is added to both Portworx Enterprise and Portworx Essentials with the Kubernetes secrets permissions.| 
| PWX-19220 | The output of `pxctl clouddrive` commands did not provide cloud-storage details making it difficult to troubleshoot issues using vSphere GUI.<br><br>**Resolution:** The outputs of the commands `pxctl clouddrive list` and `pxctl clouddrive inspect` now include vSphere datastore information and drive-labels, respectively.|
| PWX-27170 | Cloudsnap restores were not forward compatible, meaning an older version of Portworx could not restore a newer version of cloudsnap. However, in such cases, cloudsnap restore was completed without an error, but without data.<br><br>**Resolution:** Now from version 2.12.0, Portworx has a check that fails during such restore operations|



### Known issues (Errata)

| **Issue Number** | **Issue Description** |
| ---- | ---- | 
| PD-1619 |  As a result of deleting application pods that are using sharedv4 or sharedv4 service volumes in Kubernetes, part of the pod's state may not be properly cleaned up. Later, if the pod's namespace is deleted, the namespace may be stuck in the `Terminating` state.<br><br>**Workaround:** Contact {{<companyName>}} support team to clean up such namespaces.|
| PD-1611 | When using the PX-StoreV2 datastore, running multiple concurrent resize and clone operations on the same fastpath volume may cause either resize or clone operation to fail.<br><br>**Workaround:** Retry the failed operation. |
| PD-1595 | When using the PX-StoreV2 datastore, a pool may not automatically transition into `Online` state after completing a drive add operation. <br><br>**Workaround:** Perform a maintenance cycle on the node.|
| PD-1592 | When using the PX-StoreV2 datastore, pool maintenance enter or exit operation may get stuck if there are encrypted PVCs attached on the node with an outstanding IOs.<br><br>**Workaround:** Reboot the node where encrypted PVC is attached.|
| PD-1650 | When using the PX-StoreV2 datastore, a PX-Fast volume may get attached in an inactive fastpath state because of internal sanity check failure.<br><br>**Workaround**: Restart the application pod consuming the volume so that it goes through a detach and attach cycle which will reattempt fastpath activation. |
| PD-1651 | When using the PX-StoreV2 datastore, in case of failure during installation, Portworx may get into a restart loop with an error message: `PX deployment failed with an error "failed to create MD-array:”`.<br><br>**Workaround**: Clean up the failed install using node-wiper and retry installation.
| PD-1655 | You experience telemetry pod crashing issues due to port conflicts.<br><br>**Workaround:** Adjust the Portworx start port by adding the following to your StorageCluster spec:<pre>  startPort: <starting-port></pre> |


## 2.11.5

January 12, 2023

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:


| **Improvement Number** | **Improvement Description** |
| ---- | ---- |
| PD-1769 | It takes less time to upgrade Anthos because Portworx now makes fewer vSphere API calls for the following:<ul><li> To detect attached disks on each node.</li><li> To find the correct disk for attaching to a Portworx node.</li>|



## 2.11.4

October 4, 2022

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
| ---- | ---- |
| PWX-27053| Added runtime knobs to modify the amount of work that a resync workflow does at any instant of time. These knobs will facilitate throttling of resync operation.|


### Fixes

| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PWX-27028| While performing writes on the target, the resync operation pinned down resources even when the write operation had no priority to execute.<br><br>**Resolution:** In version 2.11.4, this issue is fixed.

## 2.11.3

September 13, 2022

### Fixes
| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PWX-26745 | In Portworx versions 2.11.0 through 2.11.2, Async DR restore takes longer than in previous versions.<br><br>**Resolution:** In version 2.11.3, this issue has been resolved. |
 

## 2.11.2

August 11, 2022

### New features

Portworx by Pure Storage is proud to introduce the following new feature:

* You can now enable [encryption on the Azure cloud drives using your own key](/install-portworx/cloud/azure/aks/#generate-the-specs) stored in Azure Key Vault.

### Notes

{{<info>}}**Starting with Portworx version 2.12.0, internal objectstore will be deprecated.**{{</info>}}

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
| ---- | ---- |
| PWX-26047 | `pxctl status` now shows a deprecation warning when internal objectstore is running on a cluster. |

### Fixes

| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PWX-23465 | Backups were not encrypted if `BackupLocation` in Kubernetes had an encryption key set for cloudsnaps. (Note that this should not be confused with encrypted volumes. This encryption key, if set, is applied only to cloudsnaps irrespective of encrypted volumes.)<br><br>**Resolution:** Backups are now encrypted in this case. |
| PWX-24731 | The Grafana image was not included into the list of images for the air-gapped bootstrap script. Customers using Prometheus monitoring needed to manually copy the Grafana container image into their environments.<br><br>**Resolution:** The air-gapped bootstrap script has been updated and now includes the Grafana image. |

### Known issues (Errata)

| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PD-1390 | The billing agent might try to reach outside the network portal in air-gapped environments.<br><br>**Workaround:** Disabled the call home service on Portworx nodes by running `pxctl sv call-home disable`. |


## 2.11.1

July 19, 2022

### Fixes

| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PWX-24519 | The mount path was not erased if you restarted Portworx at the wrong time during an unmount operation when using CSI. This caused pods to be stuck in the terminating state.<br><br>**Resolution:** When you restart Portworx now, it ensures that the mount path is deleted.  |
| PWX-24514 | When a cluster is configured with PX-Security and using Floating license, it was not possible to add new nodes to the Portworx cluster.<br><br>**Resolution:** You can now add new nodes to the cluster.  |
| PWX-23487 | On certain kernel versions (5.4.x and later) during startup, volume attach sometimes got stuck, preventing Portworx from starting. This is because a system-generated IO can occur on the volume while the volume attach is in progress, causing the volume attach to wait for IO completion, which in turn waits for startup to complete, resulting in a deadlock. <br><br>**Resolution:** Portworx now avoids the deadlock by preventing access to the volume until attach is complete. This functionality is only enabled after a system reboot. |


## 2.11.0

July 11, 2022

### New features

Portworx by Pure Storage is proud to introduce the following new features:

* On-premises users who want to use Pure Storage FlashArray with Portworx on Kubernetes can [provision and attach FlashArray LUNs as a Direct Access volume](/operations/operate-kubernetes/storage-operations/create-pvcs/pure-flasharray/).
* The [CSI topology feature](/operations/operate-kubernetes/cluster-topology/csi-topology/) allows users of FlashArray Direct Access volumes and FlashBlade Direct Access filesystems to direct their applications to provision storage on a FlashArray Direct Access volume or FlashBlade Direct Access filesystem that is in the same set of Kubernetes nodes where the application pod is located.
* You can now [use Portworx with IBM cloud drives](/install-portworx/cloud/ibm/) on VPC Gen2 infrastructure. Portworx will use the IBM CSI provider to automatically provision and manage its own storage disks. 
* You can enable [pay-as-you-go billing](/operations/licensing/portworx-enterprise/pay-as-you-go-air-gapped) for an air-gapped cluster with no outbound connectivity by acquiring a pay-as-you-go account key from Portworx. This key can be used on any cluster to activate the license, provided you can report usage collected by the metering module.
* You can now deploy Portworx in [IPv6](/operations/operate-kubernetes/ipv6/) networking enabled environments.

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
| ---- | ---- |
| PWX-24195 | Portworx supports using BackupLocation CR with IAM policy as an AWS s3 target for cloudsnaps triggered through Stork using ApplicationBackup CR. When Portworx detects that a BackupLocation is provided as a target, it uses the IAM role of the instance, where it is running, for authentication with s3. |
| PWX-23392 | Updated the Portworx CSI driver to CSI 1.6 spec. |
| PWX-23326 | Updated CSI Provisioner, Snapshotter, Snapshot Controller, Node Driver Registrar, and Volume Health controller to the latest releases. |
| PWX-24188 | A warning log is removed, which was printed when Docker inquired about a volume name that Portworx could not find. |
| PWX-23103 | Added a detailed warning message for volume provision issues when there are conflicting volumes in the trashcan. Volume provisioning with volume placement rules can be blocked by matching volumes in the trashcan. The new warning message informs you which trashcan volumes are causing a conflict. |
| PWX-22045 | Portworx now starts faster on high-scale clusters. During the Portworx start-up process, you will see a reduction in API calls to cloud providers. In particular, AWS API calls related to EBS volumes will be reduced. |
| PWX-20012 | Enabled support for `pd-balanced` disk type on Google Cloud environment. You can now specify `pd-balanced` as one of the disk types (such as, `type=pd-balanced, size=700`) in the device spec file.  |
| PWX-24054 | `pxctl service pool delete --help` command is enhanced to show Note and Example in addition to usage information. For example,<br>`Note: This operation is supported only on on-prem local disks and AWS cloud-drive` <br>`Examples: pxctl service pool delete [flags] poolID ` |
|PWX-23196| You can now configure the number of retries for an error from the object store. Each of these retries involves a 10-second backoff delay, followed by progressively longer delays (incrementing by 10-second intervals) between each attempt. If the object store has multiple IP addresses as the endpoints, then for a given request, the retries are done on each of these endpoints. For more details, refer to [Configure retry limit on an error](/reference/cli/cluster#configure-retry-limit-on-an-error) |
| PWX-23408 | Added support to migrate PSO volumes into Portworx through the PSO2PX migration tool.  |
| PWX-24332 | The Portworx diags bundle now includes the output of the `pxctl clouddrive list` command when available.    |
| PWX-23523 | CSI volume provisioning is now distributed across all Portworx nodes in a cluster providing higher performance for burst CSI volume provisioning. |
|   PWX-22993   |   Portworx can now be activated in PAYG (pay-as-you-go)/SAAS mode using the command<br>`pxctl license activate saas --key <pay_key>`.  |
|   PWX-23172   |   Added support for cgroups-v2 host configurations, running with docker and cri-o container runtimes. |
|   PWX-23576   |   Renamed the `PX-Essentials FA/FB` license SKU to `Portworx CSI for FA/FB` SKU.  |
|   PWX-23678   |   Added support for `px-els` (Portworx Embedded License Server) to install and operate in IPv6 network configurations. |
| PWX-23179 | Provides a way to do an in-place restore from a SkinnySnap. Now you can create a clone from a SkinnySnap and use the clone to restore the parent volume. |



### Fixes

The following issues have been fixed:

| **Issue Number** | **Issue Description** |
| ---- | ---- |
|   PWX-14944   |   Removed invalid tokens from PX-Security audit logs. When a non-JWT or invalid token was passed to the PX Security authentication layer, it was being logged. There is no impact to the user with this change.  |
|   PWX-22891   |   For On-premise Portworx installations not using cloud drives, a KVDB failover could potentially fail since Portworx on the node fails to find the configured KVDB device since its name had changed after the initial install.  <br><br>**Resolution:** Portworx will now fingerprint all the KVDB drives provided on all nodes regardless of whether they will run KVDB or not at install time. This ensures a KVDB failover happens even if device name changes on certain nodes. |
|   PWX-22914   |   When undergoing an Anthos/EKS/GKE upgrade, Portworx could experience excessive delays due to internal KVDB failovers. <br/><br/>**Resolution:** The KVDB failover rules now no longer consider storageless nodes as a KVDB candidate unless they have a dedicated `kvdb-storage` or are labeled with `px/metadata-node=true`. |
|   PWX-23018   |   In a DR setup, destination clusters sometimes ended up with volumes in the trashcan if the trashcan feature was enabled. Users had to either delete the volumes from the trashcan in the destination cluster or disable the trashcan feature in the destination cluster. |
|   PWX-23185   |   Cloudsnap restore operations sometimes slowed if a rebalance started on the same pool. <br/><br/>**Resolution:** Portorx now avoids rebalancing volumes being restored from cloudsnaps.   |
|   PWX-23229   |   If the IP address of a node changed after reboot with an active cloudsnap, the cloudsnap operation failed with "Failed to read/write extents" error. This resulted in a failed backup, and users needed to reissue the cloudsnap.<br><br>**Resolution:** Reattach the snapshot after node restart to avoid a snapshot being in an incorrect attachment state.     |
|   PWX-23384   |   When using a sharedv4 service volume with NFSv4 (default), users had to configure the `mountd` service to run on a single port even though NFSv4 does not use `mountd`. <br/><br/>**Resolution:** Portworx now skips the check for the `mountd` port when using NFSv4 and omits the `mountd` port from the Kubernetes service and endpoints objects.  |
|   PWX-23457   |   Portworx showed Pure volumes separately in the `pxctl license list` output even when there were no explicit limits for Pure volumes, which was confusing. <br/><br/>**Resolution:** This output has been improved. |
|   PWX-23490   |   Previously, upgrading from Portworx versions 2.9.0 up to anything before 2.11.0 did not update the decision matrix. <br/><br/>**Resolution:** Starting with 2.11.0, Portworx will now update the decision matrix at boot-time.  |
|   PWX-23546   |   The `px-storage` process restarted if the write complete message was received after a node restarted.    |
|   PWX-23623   |   Portworx logged benign warnings that the container runtime was not initialized.  <br/><br/>**Resolution:** Portworx no longer logs these warnings.   |
|   PWX-23710   |   When Kubernetes was installed on top of the containerd container runtime, the Portworx installation may not have properly cleaned up the `containerd-shim` process and container directories.  As a consequence, nodes may have needed to be rebooted for the Portworx upgrade process to complete.  |
|   PWX-23979   |   KVDB entries for deleted volumes were not removed from KVDB. As a result, KVDB sizes might have increased in some cases where volumes were constantly being deleted and scheduled snapshot and cloudsnaps are configured.   |
|   PWX-24047   |   Portworx will no longer use Kubernetes DNS (originally introduced with PWX-22491). In several configurations, Kubernetes DNS did not work properly. <br/><br/>**Resolution:** Portworx now relies on a more stable host's DNS config instead.   |
|   PWX-24105   |   In rare cases, Portworx on a node may have repeatedly restarted because of a panic due to nil pointer dereference when deleting a pod for a sharedv4 volume.<br><br>**Resolution:** Portworx will not come up until the relevant pod is deleted manually from Kubernetes by scaling down the application. |
|   PWX-24112   |    IBM `csi-resizer` would sometimes crash when resizing volumes.<br><br>**Resolution:**This issue is addressed in the IBM Block CSI Driver version 4.4. Check the version on your cluster using the command<br>`ibmcloud ks cluster addon ls --cluster <cluster-id>`.   |
|   PWX-24187   |   The PAYG (pay-as-you-go) license disables the license when there are issues with reporting/billing Portworx usage. After reporting/billing gets reestablished, the license is automatically enabled. Portworx did not add the default license features, requiring a restart of the `portworx.service` to properly re-establish the license.  |
|   PWX-24297   |   Portworx updated the decision matrix in the config map, causing nil pointer exceptions to appear in non-Kubernetes environments. <br><br>**Resolution:** Portworx now checks that the config map exists before updating it.    |
|   PWX-24433   |   When Docker or CRI-O is not initialized on a cluster, Portworx would periodically print the following log line: `Unable to list containers. err scheduler not initialized`.<br><br>**Resolution:** The log line is now suppressed when Portworx detects that there is runtime like Docker or CRI-O that is not initialized.    |
|   PWX-24410   |   DaemonSet YAML installs using private container registry server were using invalid image-paths (incompatible with air-gapped, or PX-Operator), thus resulting in a failure to load the required images.<br><br>**Resolution:** Fixed regression introduced with Portworx version 2.10.0, when custom container registry server was in use.    |
|   PWX-22481   |   A pod could take upwards of 10 mins to terminate if a Sharedv4 service failover and a namespace deletion happens at the same time.<br><br>**Resolution:** Scale down the pods to 0 before deleting the namespace.   |
| PWX-22128 | Async DR creates new volume (clone from its previously downloaded snapshot) on the destination cluster every time the volume is migrated. If cloudbackups are configured for the volumes from destination cluster, then every backup for a volume results in being full, as these volumes are newly created on every migration for the same volume. This change fixes this issue and allows the backups from destination cluster to be incremental. <br/><br/>**Resolution:** When a volume is migrated, the volume is in-place restored to its previously downloaded snapshot and the incremental diff is downloaded to the volume without creating any new volume. Since there is no new volume being created, any backups for this volume can now be incremental. |



### Deprecations

The following features have been deprecated:

* Legacy shared volumes 
* Volume groups
* Hashicorp Consul support

### Known issues (Errata)

| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PD-1325 | On IBM cloud drive, if `pxctl sv pool expand` with a resize-disk operation fails due to an underlying IBM issue, you will see the following error signature: `Error: timed out waiting for the condition`. This indicates that the IBM provider failed and could not perform the operation within 3 minutes.<br><br>**Workaround:** If the underlying disk on the host has expanded, then issue the command `pxctl sv pool update --resize --uid <>` to complete the pool expand operation. If the underlying disk on the host have not expanded, check the IBM csi-controller pods for any potential errors reported by IBM. |
| PD-1327 | On IBM clouddrive, if a `pxctl service pool expand -s <target-size>` with resize-disk for a target size `X` fails, you cannot issue another pool expand operation with a target size lower than the value `X`. On IBM clouddrive, a resize of the underlying disk is issued by changing the size on the associated PVC object. If IBM csi-driver fails to act upon this PVC size change, the pool expand operation will fail, but the PVC size cannot be reduced back to older value. You will see the following error: `spec.resources.requests.storage: Forbidden: field can not be less than previous value`.<br><br>**Workaround:** When a pool expand operation and a subsequent IBM PVC resize is triggered, it is expected by the IBM CSI resizer pod to eventually reconcile and complete the resize operation. Once the underlying disk on the host has expanded, then issue the command `pxctl sv pool update --resize --uid <>` to complete the pool expand operation. If the underlying disk on the host has not expanded, check IBM csi-resizer pods for any potential errors reported by IBM. |
| PD-1339 | When a Portworx storage pool contains a repl 1 volume replica, pool expansion operations report following error: `service pool expand: resize for pool <pool-uuid> is already in progress. found attached volumes: [vol3] that have it's only replica on this pool.Will not proceed with pool expansion. Stop applications using these volumes or increase replicas to proceed. resizeType=RESIZE_TYPE_ADD_DISK,skipWaitForCleanVolumes=false,newSize=150`.<br><br>The actual reason of failure is not `resize for pool <pool-uuid> is already in progress`; the correct reason of failure is `found attached volumes: [vol3] that have it's only replica on this pool.Will not proceed with pool expansion. Stop applications using these volumes or increase replicas to proceed. resizeType=RESIZE_TYPE_ADD_DISK,skipWaitForCleanVolumes=false,newSize=150`.<br><br>**Workaround:** The command `pxctl sv pool show` displays the correct error message. |
| PD-1354 | When a PVC for a FlashArray DirectAccess volume is being provisioned, Portworx makes a call to the backend FlashArray to provision the volume. If Portworx is killed or crashes while this call is in progress or just before this call is invoked, the PVC will stay in a `Pending` state forever.<br><br>**Workaround:** For a PVC which is stuck in `Pending` state, check the events for an error signature indicating that calls to the Portworx service have timed out. If such a case arises, clean up the PVC and retry PVC creation. |
| PD-1374 | For FlashArray volumes, resizing might hang when there is a management connection failure.<br><br>**Workaround:** Manually bring out the volume from the maintenance mode.  |
| PD-1360 | When a snapshot volume is detached, you see the `Error in stats    :  Volume does not have a coordinator` error message. <br><br>**Workaround:** This message appears because the volume is created, but not attached or formatted. A coordinator node is not created until a volume is attached. |
| PD-1388 | the Prometheus Operator pulls the wrong Prometheus image. In air-gapped environments, Prometheus pod deployment will fail with an `ImagePullBackOff` error. <br/><br/>**Workaround:** Before installing Portworx, upload a Prometheus image with the `latest` tag to your private registry. |
 
## 2.10.4 

Nov 8, 2022

* This version addresses security vulnerabilities.

## 2.10.3

June 30, 2022

### Improvements

| **Improvement Number** | **Improvement Description** |
| ---- | ---- |
| PWX-23523 | CSI Volume Provisioning is now distributed across all Portworx nodes in a cluster. Large volume provisioning performance increases should be seen for large enough volumes. There is no user impact for customers other than higher performance for burst CSI volume provisioning. |

## 2.10.2

June 1, 2022

### Fixes

| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PWX-23364 | Fixed a CSI volume provisioning issue where orphaned Portworx volumes were left behind if a PVC deletion was issued before volume creation finished.<br/><br/>**Resolution:** Users should upgrade their Portworx version if they are seeing orphaned Portworx CSI volumes with no associated PVC/PV. |


## 2.10.1

May 5, 2022

### Fixes

| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PWX-19815 | `pxctl credentials create` commands were failing due to an RSA error when using Google Cloud KMS as the secret provider and trying to store credentials which were too long for the RSA key to handle.<br/><br/>**Resolution:** This patch adds the fix for the issue without changing the existing operation to add credentials. |

## 2.10.0

April 7, 2022

### New features

Portworx by Pure Storage is proud to introduce the following new features:

* The Portworx [Application Control](/operations/operate-kubernetes/storage-operations/io-throttling/) feature provides a method for controlling an individual Portworx volume’s IO or bandwidth usage of backend pool resources. Portworx volumes are created from a common backend pool and share their available IOPS and bandwidth amongst all other provisioned Portworx volumes.
* The [volume trash can](/reference/cli/trashcan/) feature provides protection against accidental or inadvertent volume deletions which could result in loss of data. In a clustered environment such as Kubernetes, unintended deletion of a PV or a namespace will cause volumes to be lost. This feature is recommended in any environment which is prone to such inadvertent deletions, as it can help to prevent data loss.
* You can enable [automatic filesystem trimming](/reference/cli/volume-trim/#automatic-filesystem-trim-operations) (auto fstrim) at the volume, node, or cluster level. When you enable auto fstrim at the cluster or node level and enable `nodiscard` on your volumes, auto fstrim monitors the unused space in all filesystems mounted on Portworx volumes and automatically triggers a trim job to return unused space back to the pool, and you do not have to manually issue trim jobs.


### Notes
{{<info>}}
**Portworx 2.10 is the last release where Ubuntu 16.04 will be supported.**
{{</info>}}

### Improvements

| **Improvement Number** | **Improvement Description** |
| ---- | ---- |
| PWX-20674 | In Async DR setup, before creating a cluster pair, the DR license is checked on both the clusters. The cluster pair request will error out if only one of the clusters has DR license set. |
| PWX-21318 | Users can set the frequency of full backups using `pxctl cluster options update -b <number>`. The default value is 7, and you can check the value that is set with `pxctl cluster options list`. The number controls the number of incremental backups made before a full backup is done. |
| PWX-21024 | This fix adds `--secret_key` and `--secret_options` that allows users to propagate Kubernetes secret config information to the cli and the backend during volume import.  |
| PWX-19780 | Local volumes that are pending due to ha-increase will now appear when using `pxctl volume list --node <node>`. |
| PWX-20210 | Added support for specifying throughput parameter for gp3 drive types in AWS. The throughput parameter can only be specified at install time. Portworx currently does not allow a way to change the throughput parameter once installed. One can still change the throughput of any drives directly from the AWS console. |
| PWX-22977 | Sometimes cloudsnaps can fail with `InternalServerError`, resulting in cloudsnap backup failures and the need for user intervention to reissue the cloudsnap command for the same volume. This fix increased the aws-sdk retries and also added back-off retries. |
| PWX-21122 | VPS users now have control at the pool level with the new built-in `topologyKey` `portworx.io/pool` to allow volume affinity and anti-affinity to work for individual pools. Users can now control volume placement topology at the narrower level of pool. This allows finer control than the default topology of nodes. |
| PWX-20938 | Added a VolumePlacementStrategy template for use with StatefulSets that allows volume affinity and anti-affinity with volumes belonging to the same StatefulSet pod. Use the key/value `px/statefulset-pod"/"${pvc.statefulset-pod}` in a `matchExpression`. With this, you can ensure volumes in the same StatefulSet pod _do_ or _do not_ land on the same node. |

### Fixes

| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PWX-9712 | Some applications might starve Portworx from Async IO event reservations. This will result in a panic loop.<br/><br/>**Resolution:** The absence of Async IO reservation is now a soft failure. |
| PWX-20675 | ClusterPair can also be set up using a backup location Kubernetes object instead of creating credentials on both clusters. There was an issue where destination credentials were reset, then the system ignored the backup location object and used the internal object store. This caused migration failures.<br/><br/>**User impact:** Migration start failing for cluster pair configured using backup location.<br/><br/>**Resolution:** Resets of credentials on DR sites have been fixed for cluster pairs configured using backup location. |
| PWX-21143 | Portworx POD (oci-monitor container) was using a broad `privileged:true` security privilege, enabling too many security attributes.<br/><br/>**Resolution:** We have replaced the broad `privileged:true` security setting with fine-grained security privileges. |
| PWX-21358 | When a Portworx cluster is created in a vSphere environment, Portworx disks (vmdks) were unevenly placed among the datastores in a vSphere storage cluster. In extreme cases, all vmdks would land up in the same datastore. We have taken a best effort approach of distributing vmdks as evenly as possible among all the datastores in a Storage Cluster to the extent that vSphere apis allow.<br/><br/>**User impact:** Users had to deal with an uneven distribution of vmdks because vmdk movement across datastores is not supported. To work around this issue, users would bring up nodes one at a time.<br/><br/>**Resolution:** This best-effort approach is available for a more even distribution of vmdks among datastores of a storage pod. |
| PWX-21389 | When external EtcD v3 configured with user AuthN (authentication) as a KVDB, Portworx was not installing correctly.<br/><br/>**Resolution:** When user-AuthN is in use, KVDB clients are now properly initialized and set up. |
| PWX-21514 | In an internal kvdb cluster operating at maximum kvdb cluster size, if one of the kvdb member nodes goes down, it will be replaced by an available non-kvdb node in the Portworx cluster. When the previous member recovers from the problem and comes up, its kvdb disk will be deleted. In a vSphere environment, this deletion used to fail and users would see an additional kvdb drive when they list all available drives in the Portworx cluster. <br/><br/>**User impact:** vSphere environments could see unused kvdb disks lingering around in the cluster until they are deleted from outside of the Portworx environment.<br/><br/>**Recommendation (optional):** If users choose not to upgrade, they will have to manually delete those extra lingering disks. |
| PWX-21551 | When a Portworx volume switched to read-only mode, Portworx restarted docker-containers that use px-volumes, but it did not restart containerd/cri-o containers.<br/><br/>**Resolution:** Portworx now also restarts containerd/cri-o containers. |
| PWX-22001 | Volume placement strategy rules with `affected_replica` rules will now be applied when increasing the HA level of a volume.<br/><br/>**User impact:** Rules with `affected_replica` volume placement rules were not correctly applied when increasing HA level as they were when initially provisioning a volume for the same HA level. |
| PWX-22035 | If a node restarted when Portworx creates a snapshot after deleting a volume, Portworx sometimes restarted.<br/><br/>**User impact:** When the following things happen together:<ul><li>Users were taking a snapshot, and the snapshot operation was in the pending state</li><li>A node restarted and the parent volume was deleted during that time</li></ul>The node might have experienced a Portworx service restart.<br/><br/>**Resolution:** This release addresses this issue and Portworx no longer restarts under this circumstance. |
| PWX-22478 | Portworx `node-wipe` operation did not clean up all the old node identitifiers, which caused issues with the telemetry container after the node was wiped or recycled.<br/><br/>**Resolution:** Portworx `node-wipe` procedure was fixed, so all node identities are properly recycled. |
| PWX-22491 | Portworx installations were using the default `dnsPolicy`, which did not include the Kubernetes internal DNS server.<br/><br/>**Resolution:** We changed the default `dnsPolicy` to `ClusterFirstWithHostNet`, so now the Kubernetes DNS is also used in hostname resolution. |
| PWX-22787 | Portworx generates a core then restarts on certain nodes where application pods are trying to setup.<br/><br/>**User impact:** Portworx will generate a core and restart itself in the scenario where an application pod is trying to attach a volume on a node and the volume is already attached and busy in another node in the cluster. The Portworx service will auto-recover from this after the restart. Only Portworx 2.9.1 was impacted by this issue.<br/><br/>**Resolution:** The issue causing Portworx to restart has been fixed. |
| PWX-22791 | There is no update API for credentials.<br/><br/>**User impact:** When keys are rotated for cloudsnap credentials, there is no way to update the credentials with the new keys. Only way is to delete and recreate credentials with new keys. This requires stork schedule for cloudsnap to be updated with new credential ID to avoid failures due to credential ID mismatch. With porx version 2.10, update API has been added to credentials and allows users to update most of the parameters. Caution must be exercised while changing parameters such as bucket or the end point, which causes previous cloudsnaps to be no longer visible through the modified credentials.  |
| PWX-22887 | After a node is decommissioned, backups may fail for volumes which were attached on the decommissioned node.<br/><br/>**Resolution:** Detaching the volume using `pxctl host detach` fixes the issue.  |
| PWX-22941 | While performing an internal kvdb node failover, a failure in setting up internal kvdb could result into an orphaned unstarted node entry in the internal kvdb cluster.<br/><br/>**User impact:** Internal kvdb clusters would keep running at a reduced cluster size.<br/><br/>**Resolution:** A Portworx node which failed to perform internal kvdb failover will detect its own orphaned node entry in the kvdb cluster and clean it up.  |
| PWX-22942 | In stretch cluster, cloudsnaps can sometime fail with `Not authenticated with secrets` error. This is due a missing check, which schedules the cloudsnaps on a node in a Kubernetes cluster without access to credentials (BackupLocation).<br/><br/>**Resolution:** Fix the check to not allow cloudsnaps on nodes without Kubernetes secrets. |
| PWX-23060 | Previously, FlashArray and FlashBlade had a limit of 200 Portworx or FA/FB volumes in a cluster.<br/><br/>**Resolution:** The limits are now 200 Portworx and 100,000 FA/FB volumes in a cluster. |
| PWX-23085 | A Portworx upgrade on a storageless node in the cloud drive configuration can get stuck with the message `DriveAttachedOnDifferentNode Error when pool ExpandInProgress on another node in the cluster`.<br/><br/>**Resolution:** The issue has been fixed. Portworx now skips drives where expansion is in progress until the expansion is complete. |
| PWX-23096 | Cloudsnaps may stay in active state forever when the executing node is decommissioned.<br/><br/>**User impact:** Cloudsnaps may be stuck in active state for ever. Further requests to the same volume may be in queued state.<br/><br/>**Resolution:** Mark that these cloudsnaps are stopped, allowing newer requests run on a different replica. |
| PWX-23099 | Restoring cloudsnap with the intention of doing inplace restore fails with the error `Not enough space available on the pool`, even though the used size of the volume being restored is less than available space on pool.<br/><br/>**User impact:** Failure to restore the cloudsnap to the same pool as parent volume.<br/><br/>Resolution: Check for pool space with respect to used size of the volume being restored than the actual volume size. |
| PWX-23119 | Panic was occurring in one of the Portworx processes when creating a local snapshot for replicated volumes having more than 1 replica and which have the skinnysnap feature enabled.<br/><br/>**User impact:** When the skinnysnap feature is enabled and the number of skinnysnaps is set to greater than 1, the Portworx process on one or more nodes may panic while creating a local snapshot. This happens if not all replicas are online on a volume with a replication level greater than 1.<br/><br/>**Resolution:** Fix the out of bounds access that caused the panic in skinnysnap creation path with replicated volumes having more than 1 replica. |
| PWX-23151 | On OpenShift platform, the Portworx service could not use Kubernetes client APIs if the Portworx POD was stopped.<br/><br/>**Resolution:** The Portworx service has been isolated from Portworx POD, so stopping the POD on OpenShift platform no longer prevents Portworx service from using Kubernetes client APIs. |
| PWX-23155 | In a cluster with more than 512 nodes, Portworx fails to start after 511 nodes. Portworx had a limit to open client connections which gets crossed while adding nodes after 511.<br/><br/>**User impact:** More than 511 nodes cannot be added to the cluster or it will cause Portworx to be in a crash loop. |
| PWX-23174 | For fragmented large volumes, `repl-add` keeps restarting from scratch.<br/><br/>**User impact:**<br/>As `repl-add` is stuck in loop, `ha-add` won't finish, and a new replica is not added to the replica set. |
| PWX-23189 | On Tanzu vSphere with the Kubernetes platform, the worker VMs have a 16GiB root partition. Due to the small size of the root partition, this can lead to disk pressure as the life of the cluster increases.<br/><br/>**Resolution:** We recommend that you monitor the free disk space on these workers continuously and garbage collect space as needed.  |

### Known issues (Errata)

| **Issue Number** | **Issue Description** |
| ---- | ---- |
| PD-1104 | When installed in vSphere local mode, if a VM which is running a storageless Portworx node is migrated to another ESXi which does not have any storage Portworx nodes, this storageless Portworx node will fail to transition into a storage node.<br/><br/>**Workaround:** Restarting Portworx on the storageless Portworx node will transition it into a storage node. Portworx can be restarted by applying the `px/service=restart` label on the Kubernetes node or by issing `systemctl restart portworx` on the node. |
| PD-1117 | When trash can is enabled in disaster recovery setup (by setting a value greater than 0 for `VolumeExpiration` in cluster settings) on the destination cluster,  users will see many volumes. If the expiration is set to a very large number, these snapshots might take up significant capacity as well. This is a known issue and will be addressed in future release.<br/><br/>**Workaround**: Do not enable `VolumeExpiration` in cluster settings. |
| PD-1125 | If a Portworx storage pool is in an `Error` state (seen in `pxctl service pool show`), do not submit new pool expand operations on the pool.<br/><br/>**Workaround**: Before submitting new pool expand operations, fix the pool state by entering and then exiting the pool in maintenance mode using the `pxctl service pool maintenance` command.  |
| PD-1127 | Portworx pool expand operation has the status `failed to update drive set state: etcdserver: leader changed`.<br/><br/>**Workaround:** This error indicates that the actual pool expansion is complete in the background. The message occurs when Portworx tries to update the status of the drives in the pool. |
| PD-1130 | A storage pool expand using `add-disk` can be stuck in progress with the error `Pool is still not up. add drive status: Drive add: No pending operation pool status: StorageFull`.<br/><br/>**Workaround:** Restart Portworx on the node to resolve the issue. This can be done by issuing `systemctl restart portworx` or labeling the Kubernetes node with the `px/service=restart` label. |
| PD-1165 | Due to an incomplete container image, Portworx installation or upgrade operations can get stuck with the message: `could not create container: parent snapshot <> does not exist: not found`.<br/><br/>**Workaround:** Identify the `px-enterprise` image and remove it. The following sample commands do this:<br/><br/><code>ctr -n k8s.io i ls \| grep docker.io/portworx/px-enterprise:2.10.0</code> <br/><br/><code>ctr -n k8s.io i rm docker.io/portworx/px-enterprise:2.10.0</code>|


## 2.9.1.4

Apr 1, 2022

### Notes

* This version addresses security vulnerabilities.

## 2.9.1.3

Mar 15, 2022

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-22943 | Portworx with FA cloud drives was erroneously able to start with the `user_friendly_names` setting enabled. <br/><br/>**User impact:** Portworx installs successfully initially, but on restart it won't be able to identify its own drives. This could cause Portworx to create new drives ignoring the already created ones. <br/><br/>**Resolution:** Portworx no longer starts if the multipath `user_friendly_names` setting is enabled. If after installing this version you receive this error, update your multipath configuration. |

## 2.9.1.1

Feb 3, 2022

{{<info>}}
**ADVISORY:** Pure Storage strongly recommends users of the 2.9.1 release upgrade to 2.9.1.1.
{{</info>}}

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-22787 | Under a certain race condition, Portworx could generate a core and restart itself. This could happen when an application pod tries to attach a volume on a node while the volume is already attached on another node in the cluster.  <br/><br/>**User impact:**  Portworx on the node where the application pod is trying to attach the volume would generate a core and restart. The Portworx service auto-recovered from this after the restart. Only Portworx 2.9.1 was impacted by this issue.  <br/><br/>**Resolution:** The issue causing Portworx to restart has been fixed. |

## 2.9.1

Jan 27, 2022

### New features

Portworx by Pure Storage is proud to introduce the following new features:

* Support for [Pure FlashBlade as a Direct Access filesystem](/operations/operate-kubernetes/storage-operations/create-pvcs/pure-flashblade) has graduated from early access to Generally Available! With this feature, Portworx directly provisions FlashBlade NFS filesystems, maps them to a user PVC, and mounts them to pods. Reach out to your account team to enable this feature. 
* Support for [Pure FlashArray cloud drives](/cloud-references/auto-disk-provisioning/pure-flash-array/) has graduated from early access to Generally Available! Use FlashArrays as a cloud storage provider. Reach out to your account team to enable this feature. 

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-22105 | Portworx now supports PKS distributions based on "containerd" container runtimes. |
| PWX-21721 | The `pxctl status` command's response time is now reduced when telemetry is enabled. This was done by running telemetry status asynchronously and caching its status. |
| PWX-20642 | Portworx no longer requires global permissions on all datastores. Users can now specify which datastores to give Portworx access to. |
| PWX-22195 | Improved Portworx logs by adding a co-relation ID to every API request and will be logged at all levels. |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-22197 | CSI provided drives may have incorrect classification of media type: Disks based on flash media getting classified as Magnetic disks. <br/><br/>**User impact:** Portworx may incorrectly classify other storage attributes which are derived from the storage media like IO Priority, etc. <br/><br/>**Resolution:** CSI provided drives are now correctly classified. If you’ve had your drives incorrectly classified, you can manually change the io_priority of the pool using the `pxctl sv pool update` command. |
| PWX-22079 | A golang panic (stacktrace) occurred when there was an error initializing the storage layer.<br/><br/>**User impact:** When there was an error while initializing the storage-layer, golang sometimes panicked (stacktrace output), and the real error was masked.<br/><br/>**Resolution:** Portworx now properly handles errors from the storage layer, and no longer causes golang panics. |
| PWX-21605 | Portworx keeps a track of the number of NFS threads configured on a node. If the number of threads drops below 80% of the configured value it will reset it to the configured thread count. However a variance of 80% was too large, and on an overloaded system could cause the system to run with fewer number of threads than desired. <br/><br/>**User impact:** On certain overloaded systems, sharedV4 application pods could see NFS timeouts since the NFS server had less number of threads than the configured amount. <br/><br/>**Resolution:** Portworx will now keep the NFS thread count within 95% of the configured value. |
| PWX-22313 | Transitioning a storageless node into a storage node caused other nodes in the cluster to receive a NodeDown event (for the storageless node) with an IP which matches with the new storage node. This caused the sharedV4 server to assume that there were no sharedV4 clients active on that IP. <br/><br/>**User impact:** A sharedV4 app running on such a node which transitions could see I/O errors.<br/><br/>**Resolution:** Portworx on peer nodes will now detect these transitions, ignore NodeDown events for the same duplicate IP, and avoid removing the client for the sharedV4 volumes. |
| PWX-22244 | When `fsGroup` is set on the volume, the kubelet has to perform a recursive permissions change on the mount path. This can take time and delay the pod creation when there are a large number of files in the volume. In Kubernetes 1.20 or later, there is a setting `fsGroupChangePolicy: OnRootMismatch` which tells kubelet to skip the recursive permissions change if the permissions on the root (mount path) are correct. This prevented the delay in pod creation, but was rendered ineffective when Portworx reset a permissions value. <br/><br/>**User impact:** Specifying `fsGroupChangePolicy: OnRootMismatch` did not alleviate the pod creation delay caused by the `fsGroup` setting. |
| PWX-22237 | A race condition in Portworx during storageless node initialization sometimes created an orphaned entry in `pxctl clouddrive list` where that node's entry was not present in `pxctl status`. <br/><br/>**User impact:** The node list between `pxctl status` and `pxctl clouddrive list` was not in sync. <br/><br/>**Resolution:** Portworx now handles this race condition and ensures that such orphaned entries are removed.| 
| PWX-22218 | Portworx failed to mount volumes into asymmetrical shared mounts. <br/><br/>**User impact:** When using asymmetrical shared mounts (e.g. mounting different directories between host/container), it was not possible to mount Portworx volumes into these directories. <br/><br/>**Resolution:** After the fix, asymmetrical shared mounts work properly (i.e. you can mount volumes into such directories). |
| PWX-22178 | On Kubernetes installations that use the CRI-O container runtime, setting up a custom bidirectional (shared) mount for the Portworx pod did not propagate to `portworx.service`. Instead, it would be set up as a regular bind-mount, that could not be used to mount the PXD devices. <br/><br/>**Resolution:** The bidirectional mounts are now properly propagated to `portworx.service`.|
| PWX-21544 | When coming out of run-flat mode after more than 10 minutes have elapsed, a Portworx quorum node sometimes failed to start a watch on the internal KVDB because the required KVDB revision had already been compacted. <br/><br/>**User impact:** When using an internal KVDB, there may have been a brief outage when Portworx exits run-flat mode. |
 


### Known issues (Errata)

Portworx is aware of the following issues, check future release notes for fixes on these issues:

|**Issue Number**|**Issue Description**|
|----|----|
| PD-1076 | When using Portworx with FlashArray, if new drives are added while paths are down, it may not have all connections established and may result in failures when only a certain subset of paths go down, even if others are live. <br/><br/>**Workaround:** This can be recovered after all paths are present with `pxctl sv m --cycle`, which will detach and reattach the drives, hopefully ensuring all paths are added back. |
| PD-1068 | When using Portworx with FlashArray, expanding a pool by resizing when some paths are down (even if some are still up) may result in issues, as the single paths may not pick up the new path size and fail the multipath resize operation. <br/><br/>**Workaround:** Run the resize again when all paths are restored to resolve the issue and complete the expansion. |
| PD-1038 | Portworx pool expand operations fail to resize when some multipath connection paths of FA are down. <br/><br/>**Workaround:** After the network is restored, you can run `iscsiadm -m session --rescan` and expand the pool again. | 
| PD-1067 | Disabling a port on the FlashArray will also remove it from the list of ports in the REST API, and thus Portworx will not attach it. This can cause some multipath paths to remain offline even after the port is reattached, especially if Portworx had a restart while the port was down. <br/><br/>**Workaround:** You can recover the faulty paths by running the `pxctl sv m --cycle` command to reattach them and bring back the missing paths. Note that unless all paths are down, Portworx will still function fine, just with reduced iSCSI/FC-layer redundancy. |
| PD-1062 | `px-pure-secret` contains FlashArray and FlashBlade connection information, specifically management endpoint and token. The secret is loaded when Portworx starts, therefore it needs to be present before Portworx is deployed. Also, any changes to the secret after Portworx is already started will not be detected. <br/><br/>**Workaround:** If you need to change array backends or renew the token, you must restart Portworx. This also applies to FlashArray disk provisioning, and impacts changes to the FA/FB essentials licenses. |
| PD-1045 | In the cloud drive mode of deployment with a FlashArray, a restart of the primary controller or a network outage to the FlashArray could cause a storage node to transition into a storageless node. This transition happens since another storageless node in the cluster picks up the disks and starts as a storage node.  The original storage node however still ends up having the signature of the old disks and starts up as a storageless node with StorageInit failure. This happens only if Portworx on this node is unable to cleanly detach its disks due to the primary controller on FlashArray being down.<br/><br/>Similarly, after the primary controller restarts or a network outage occurs, a storage node could see errors from the backend disk, or the internal KVDB could see errors from the KVDB disk and cause Portworx or the internal KVDB on that node to enter an error state. <br/><br/>**Workaround:** Once the primary controller is back, restart Portworx on the impacted node to recover. |
| PD-1063 | If the Kubernetes ETCD is unstable, Portworx may experience intermittent access issues to the Kubernetes API. <br/><br/>**Workaround:** If a pool expand operation fails with the error message: `could not retrieve portworx-storage-decision-matrix config map: etcdserver: leader changed"`, retry the pool expand operation. |
| PD-1093 | Application pods can get stuck in the `ContainerCreating` state. Check for _both_ of the following conditions to determine if volume attachment has failed:<ul><li>When you run `kubectl describe pod`, you see the following message:<br/>`MountVolume.SetUp failed for volume "<volume-name>" failed to attach volume: Volume: <vol-id> is attached on: <node-where-vol-is-attached>`</li><li>In your Portworx logs, you see the following error:<br/>`attach_if_mounted: failed to attach dev <vol-id> (-12ev)`</li></ul>**Workaround:** Restart the Portworx service on this node. You can also schedule the app on a different node, which moves the volume attachment location. |
| PD-1071 | If you manually disconnect any connected volumes from FlashArray, the Portworx node may become stuck attempting to reconnect to the original volume if there are pending I/Os.<br/><br/>**Workaround:** Reconnecting the volume will resolve this issue at the next Portworx restart and the node will return to a healthy state. |
| PD-1095 | If you uninstall Portworx with `deleteStrategy` set to [`Uninstall`](/operations/operate-kubernetes/uninstall/uninstall-operator/) (and not `UninstallAndWipe`), then you reinstall Portworx, the telemetry service and metrics collector will be unable to push metrics and may run into a `CrashLoopBack` state. This is for certificate security reasons.<br/><br/>**Workaround:** [Contact Support](/support/) to reissue the certificate. |

## 2.9.0

Nov 22, 2021

### Notes

* If you're using Kubernetes 1.22, you should use Stork 2.7.0 with Portworx 2.9.0.
* After upgrading to Portworx 2.9.0, the existing `sharedv4 service` volumes will be switched to using NFS version 4.0.

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-18037 | Improved `pxctl status` bootstrap issue reporting when KVDB connectivity is blocked. |
| PWX-18038 | Clarified error message when using an incorrect network interface. |
| PWX-18362 | Using the `pxctl cloudsnap list -d` command, you can now list cloudsnaps of volumes that are no longer present in the cluster, but belonged to it. |
| PWX-20670 | Portworx will attempt to enable persistent journaling when installing . |
| PWX-21373 | The following template can now be used in a `VolumePlacementStrategy` for the `volumeAntiAffinity` or `volumeAffinity` to automatically constrain the `MatchExpressions` to the PVC namespace.<br\>``- key: "namespace"``<br>&nbsp;``values:``<br>&nbsp;``- "${pvc.namespace}"``<br>You can now separate interaction between different namespaces when using volume (anti-)affinity in VPS. |
| PWX-21506 | One of the folders used by legacy shared (fuse) volumes will not created unless shared volumes are created and mounted. This change prevents the internal mount path, specifically (`/opt/pwx/oci/rootfs/pxmounts`), from being created when there are no shared volumes being used. |
| PWX-21994 | Added support for the `cgroup V2 -configured` hosts. |
| PWX-21662 | Portworx now supports OpenShift version 4.9. | 
| PWX-21341 | Added the `sharedv4_failover_strategy` storageClass parameter whose value can be either `aggressive` or `normal`. The `aggressive` strategy uses a shorter failover grace period than the one used by the `normal` strategy. If `sharedv4_failover_strategy` is unspecified, then the default for sharedv4 service volumes is `aggressive` and that for sharedv4 volumes is `normal`. The value for this parameter can be changed using the `pxctl volume update` command as well. An empty value clears the setting. |
| PWX-21684 | Telemetry is now disabled by default opn the spec generator. Enable telemetry under advanced settings. |
| PWX-21895 | KDVB Metrics are now disabled by default, lowering the amount of metrics generated. You can reenable them by adding `kvdb_metrics_enable=1` as a runtime option.

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-21710 | If a KVDB node encounters IO errors and restarts, it will fail to unmount the previous mountpoint. <br/><br/>**User Impact:** The KVDB node did not start, resulting in reduced KVDB availability if there were no other nodes available to take over for the failed replica.<br/><br/>**Resolution:** The Portworx container now looks for unhealthy mount points and unmounts them. | 
| PWX-21590 | Portworx nodes would not transition into run-flat mode when the `etcd` cluster was unreachable and lost cluster quorum. Each node detected the `etcd` cluster as unreachable at different times, rendering the cluster unable to reach consensus on whether `etcd` quorum is lost. <br/><br/>**User Impact:** Cluster KVDB quorum as well as Portworx cluster quorum would be lost and Portworx would not transition into run-flat mode.<br/><br/>**Resolution:** All Portworx nodes will now detect `etcd` is not reachable within 1 minute and enter run-flat mode. |
| PWX-21506 | Mount paths used by shared (fuse) volumes have wider permissions than desired. <br/><br/>**User Impact:** One of the folders used by legacy shared (fuse) volumes were not created unless shared volumes were created and mounted.<br/><br/>**Resolution:** This change prevents the internal mount path, specifically (`/opt/pwx/oci/rootfs/pxmounts`), from being created when there are no shared volumes being used. |
| PWX-21168 | Added support for the RKE2 Kubernetes distribution. <br/><br/>**User Impact:** The RKE2 has switched to K3s Kubernetes distribution baseline, which broke the Portworx deployment.<br/><br/>**Resolution:** The YAML-generator at [Portworx Central](https://install.portworx.com) had been fixed to recognize the RKE2-based Kubernetes version, and automatically apply the customization required to install Portworx. |
| PWX-20780 | Pods using encrypted sharedv4 volumes got stuck in the terminating state. <br/><br/>**User Impact:** If a node that was hosting the replica of a sharedv4 encrypted volume (server node) was rebooted, it was possible for application pods accessing that volume to get stuck in the terminating state.<br/><br/>**Resolution:** Portworx will detect restart of a sharedv4 encrypted volume server node and will automatically restart application pods using that volume which will recover them to functional state. |
| PWX-21965 | Portworx `.stack` files were getting accumulated in `/var/cores` and not deleted. <br/><br/>**User Impact:** Users experienced a growing number of files with the `.stack` extension in `/var/cores` on the the worker nodes, which they must delete manually.<br/><br/>**Resolution:** The new `.stack` files generated in Portworx now are now deleted automatically. |
| PWX-21823 | In cloud setups, if a cluster was scaled down, resulting in internal KVDB nodes being shutdown, then there was a possibility that more than the quorum number of nodes were removed from the bootstrap configuration map where the internal KVDB nodes list is maintained. <br/><br/>**User Impact:** If internal KVDB nodes were scaled down, users had to manually recover the KVDB after scale up.  |
| PWX-21807 | Expanding a pool using `add drive` beyond 6 drives caused creation of new pools. <br/><br/>**User Impact:** If you had six drives in a single pool, and you tried the `pxctl service pool` expand command to add drives, the command created a brand new pool instead of expanding the existing one.<br/><br/>**Resolution:** The `pxctl service pool` command will now fail the operation with an error message indicating that the pool has reached the maximum number of drives. |
| PWX-21664 | When Kubernetes was installed on top of the `containerd` container environment and using an older (non-default) runtime version `io.containerd.runtime.v1.linux`, Portworx installation sometimes did not properly clean up the `containerd-shim` process and container directories. <br/><br/>**User Impact:** The node may have required a reboot for Portworx upgrade process to complete.<br/><br/>**Resolution:** Portworx now cleans up processes and directories when running on this older `containerd` runtime. |
| PWX-21659 | In certain failed scenarios where cloud backup would fail even before creating a cloud backup ID, the migration status would not be updated with the appropriate error message. <br/><br/>**User Impact:** Migrations triggered as a part of Async DR would fail with an empty error message.<br/><br/>**Resolution:** Portworx now sets the correct message in all cloud backup error scenarios. |
| PWX-21558 | You can sepcify discard or nodiscard filesystem mount options using two different volume spec fields (`volumespec.nodiscard` and `volumespec.mount_options`). <br/><br/>**User Impact:** This resulted in ambiguous volumespec settings in the following cases:<br>``volumespec.nodiscard = true and volumespec.mount_options="discard="``<br>``volumespec.nodiscard = false and volumespec.mount_options="nodiscard="``<br/><br/>**Resolution:** When ambiguous volumespec settings for discard or nodiscard are detected, Portworx uses the `volumespec.nodiscard` to derive the final value, and generates an alert to notify that the volumespec needs to be fixed. |
| PWX-21380 | One of the internal data structures was not properly protected for access from multiple threads. This caused Portworx to encounter an error and be restarted. <br/><br/>**User Impact:** In a scaled-up setup with a large number of nodes, Portworx restarted intermittently.<br/><br/>**Resolution:** Portworx now does not share the data structure between multiple threads. |
| PWX-21328 | Applications using file locks do not work with `sharedv4 service` volumes while using NFS version 3. Any attempt to acquire a file lock hangs indefinitely. <br/><br/>**User Impact:** Containers stuck in this state cannot be terminated using normal methods.<br/><br/>**Resolution:** This no longer occurs with sharedV4 service volumes. |
| PWX-21297 | The Portworx pod did not encode stored credentials correctly when authenticating with the container-registry to download the `px-enterprise` container. <br/><br/>**User Impact:** Depending on the characters used in password for the container-registry, authentication continuously failed, and the Portworx pod was unable to pull and install Portworx on the cluster.<br/><br/>**Resolution:** Portworx now properly encodes credentials. |
| PWX-21004 | Locks are not transferred on failover. <br/><br/>**User Impact:** Users saw unpredictable behavior for applications relying on NFSv4 locking.<br/><br/>**Resolution:** Portworx disallows NFSv4 for sharedv4 service volumes. |
| PWX-20643 | Pods that used several sharedv4 volumes sometimes became stuck in the terminating state. <br/><br/>**User Impact:** When a pod that was using several sharedv4 volumes was deleted, it sometimes became stuck in the terminating state. Users had to reboot the node to escape this state.<br/><br/>**Resolution:** Pods using sharedv4 volumes no longer get blocked indefinitely. |
| PWX-19144 | During migration, Portworx volume labels were not copied. <br/><br/>**User Impact:** Users were forced to manually copy volume labels after migration to reach the desired state.<br/><br/>**Resolution:** Volume labels are now copied automatically during migration. |
| PWX-21951 | The Operator created more storage nodes than `maxStorageNodePerZone` specified in STC.<br/><br/>**User Impact:** The Portworx cluster came up with a different number of storage nodes than the number specified using the `maxStorageNodePerZone` parameter in the STC.<br/><br/>**Resolution:** Portworx now comes up with the exact number of storage nodes specified in the `maxStorageNodePerZone` parameter. |
| PWX-21799 | Added support for `PX_HTTP_PROXY` with RKE2 installs. <br/><br/>**User Impact:** When installing Portworx in air-gapped environments, you can include the `PX_HTTPS_PROXY` environment variable to use http-proxy with the install. However, this variable was not used when pulling px-enterprise image during installs on Kubernetes with containerd container-runtime clusters.<br/><br/>**Resolution:** Portworx now uses the `PX_HTTPS_PROXY` environment variable when installing Portworx on Kubernetes with containerd container-runtime clusters. |
| PWX-21542 | When you installed Portworx on an air-gapped cluster, the node start up time was delayed. This happened because the metering agent ran to report the health of Portworx cluster. <br/><br/>**User Impact:** Portworx tried to run the metering agent, resulting in a delayed start.<br/><br/>**Resolution:** The metering agent is now disabled to avoid the delayed start. |
| PWX-21498 | The drain volume attachment job timeout was too long.<br/><br/>**User Impact:** The drain volume attachment job had an upper time limit of 30 minutes. In a certain error scenarios the job will stay in pending for 30 minutes and then timeout and fail.<br/><br/>**Resolution:** Changed the drain volume attachment job timeout from 30 minutes to 10 minutes. Any volume attachment drain operation is expected to complete within 10 minutes.  |
| PWX-21411 | When a Portworx node went out of quorum and then rejoined the cluster, the volume mountpoint sometimes became read-only. <br/><br/>**User Impact:** Some application pods became stuck in  the `container creating` or `crashloopbackoff` state. <br/><br/>**Resolution:** Portworx now detects the pods with ReadOnly PVC and proactively bounces the application pods after Portworx startup completes on a node. | 
| PWX-21224 | Setting cloudsnap threads to four or less resulted in cloudsnap backups in hung state. <br/><br/>**User Impact:** With cloudsnap threads (Cloudsnap maximum threads field in cluster options) set to four or below and doing more than 10 cloudsnaps at the same time, cloudsnaps became stuck and made no progress.<br/><br/>**Resolution:** Incorrect check for thread count no longer results in a deadlock. |
| PWX-21197 | The runtime option `limit_drivers_per_pool` did not work in Portworx version 2.8.0. <br/><br/>**User Impact:** The `limit_drives_per_pool` is a runtime option to control the number of drives in a pool. It was possible for the last pool to have more drives than the limit under the following circumstances: if the drive count is not an exact multiple of the limit and if creating another pool will have too few drives within. Internally, a new pool is created if drive count is at least 50% of the limit is available in the last pool.<br/><br/>**Resolution:** Now, during the later drive add from maintenance mode, these limits are honoured more strictly. Any drive add will fall into a pool only if the drive count is within the limit. If not, a new pool will be formed. |
| PWX-21163 | Prometheus was unable to access the Portworx internal ETCD on a multi network interface setup causing ETCD alerts not to appear. <br/><br/>**User Impact:** Prometheus alerts and monitoring did not work properly for the alerts related internal ETCD on multi-network interface setups.<br/><br/>**Resolution:** Portworx now allows internal `etcd` access from all network devices for Prometheus can have access and scrape alerts. |
| PWX-21057 | Pods failed to come up for restored PVCs that were encrypted with Vault namespace secrets. <br/><br/>**User Impact:** If you used a PVC that was cloned from a snapshot, and it was encrypted by Vault namespace secrets, then pods using that PVC were stuck in the container creating state.<br/><br/>**Resolution:** Portworx now checks for the Vault namespace value at the correct place in the volume spec for restored volumes. This allows pods to finish setup and not get stuck. |
| PWX-21037 | Unable to set the `mount_options` in volume spec. The `nodiscard` or `discard` volume fields were not in sync with `mount_options`.<br/><br/>**User Impact:** This resulted in unpredictable behavior wherein after mounting a `pxd volume`, volumes appeared mounted with `discard` option even when `volumespec.nodiscard` was set to `true`.<br/><br/>**Resolution:** Portworx now allows setting mount options. The `volumespec.nodiscard` and mount options are now made to be in sync, and this provides predictable behavior for `pxd volume` mounts. |
| PWX-20962 | Portworx experienced a startup issue with volatile mounts of files in Kubernetes environments.<br/><br/>**User Impact:** If files were manually removed from the `/opt/pwx/oci/mounts/` directory, followed by restarting Portworx service before Portworx pod, this sometimes resulted in a looping failure to start the Portworx service.<br/><br/>**Resolution:** Synchronization of volatile mounts into `/opt/pwx/oci/mounts/` is fixed, so the restart of the Portworx pod properly restores files/directories in this directory. |
| PWX-20949 | Cloudsnap restore operations sometimes failed with the error message `Restore failed due to stall`. This happened when the restore operation incorrectly evaluates the node to be in maintenance mode. <br/><br/>**User Impact:** User restores may not have completed, and users may have been required to restart the node where the restore was stalled, or reissue the restore command.<br/><br/>**Resolution:** Portworx cloudsnap now does not interpret node status. An upper layer module handles it, preventing this issue. |
| PWX-20942 | Synchronous DR on Tanzu cloud drives did not work. <br/><br/>**User Impact:** In Tanzu, the cloud drive backing Portworx is a PVC, which is a cluster-scoped resource. In order to help clusters distinguish between their drivesets, the drivesets should have been labeled accordingly.<br/><br/>**Resolution:** Portworx now supports Tanzu PVCs. |
| PWX-20903 | The `affected_replicas` in `VolumePlacementStrategy ReplicaPlacementSpec` were not applied correctly when multiple were used. <br/><br/>**User Impact:** Volume provisioning sometimes failed when you had a VPS replicaAffinity with multiple rules using affected_replicas that _should_ work.<br/><br/>**Resolution:** Using multiple `affected_replicas` rules now work in Portworx as expected. |
| PWX-20893 | The `pxctl v check --mode fix_safe <volname/volid>` command failed when `Deleted inode <ino> has zero dtime` was found by `fsck`. <br/><br/>**User Impact:** If a file was in use at the same time it was deleted, it was never properly closed, and the filesystem was remounted, users may have seen an error from `fsck`.<br/><br/>**Resolution:** The `pxctl v check --mode fix_safe <volname/volid>` command now recovers the volume to a clean state. |
| PWX-20546 | Portworx sometimes displayed a rebalance job status as `DONE` while rebalance actions were still pending. <br/><br/>**User Impact:** You may have seen a rebalance job status as `DONE` with some pending jobs belonging to deleted volumes.<br/><br/>**Resolution:** Portworx now does not show the rebalance job for deleted volumes. |
| PWX-21279 | In some cases, when multiple matchExpressions were specified in a volume placement strategy, a pool which satisfied only some expressions got selected. <br/><br/>**User impact:** Instead of multiple matchExpressions, users could use multiple rules in versions affected by this issue to get the expected result.<br/><br/>**Resolution:** Portworx now checks for all match expressions.  |
| PWX-20029 | If an NFS client node was restarted, the client reload process would restart all the pods assigned to the node. If it detected that an NFS server is down, it would start the timer for that event. When the timer expired, the process invoked the recovery routine, and the pods that use the volumes with that NFS were restarted again. This restart was sometimes unnecessary.<br/><br/>**User impact:** Users saw pods reset unnecessarily. <br/><br/>**Resolution:** Portworx now gets the latest list of attached volumes in the recovery routine, and only restarts the pods that are still using the stale volumes. |

### Known issues (Errata)

Portworx is aware of the following issues, check future release notes for fixes on these issues:

|**Issue Number**|**Issue Description**|
|----|----|
| PD-1015 | If the Portworx pod is deleted and recreated while a pool expand of a Portworx Pool is in progress, the pool expand will fail with the error message: `could not retrieve portworx-storage-decision-matrix config map: Unauthorized`.<br/><br/>**Workaround:** Wait for the new pod to come up and then resubmit the pool expand request. The subsequent request will go through. |
| PD-1007 | During a Portworx cluster upgrade, if the cluster is using `ContainerD` as the runtime, a Portworx node may get stuck during upgrade.<br/><br/>**Workaround:** Check if the `px-oci-installer` process is stuck on the node using the<br>`ps -ef | grep px-oci-installer` command.<br>If yes, kill the process manually using the following steps:<br>``ps -ef | grep containerd-shim | grep /px-oci-installer | awk '{print $2}' | xargs kill -1``<br>``crictl ps  ; crictl rm px-oci-installer``<br>``rm -fr /run/containerd/runc/k8s.io/px-oci-installer /run/containerd/io.containerd.runtime.v1.linux/k8s.io/px-oci-installer``<br>Alternately, you can reboot the entire node which will automatically kill the stuck process. |
| PD-1005 | In certain scenarios, a PVC resize request issued from Kubernetes can error out and the API will fail, but Portworx will eventually complete the resize operation. Kubernetes retries the operation but Portworx then returns the error: `No change is requested`. This causes the PVC size to not match with the actual Portworx volume size. |
| PD-990 | Application containers running with non-Docker container runtimes will not get automatically restarted when Portworx detects issues with the volume mounts for those containers. Examples of non-Docker container runtimes are containerd, CRI-O, rkt. The most common scenario where volume mount issues are detected is when the mount becomes read-only when Portworx is down on a node for more than 10 minutes.<br/><br/>**Workaround:** Restart application containers to workaround this issue. In Kubernetes, this means deleting application pods so that they get recreated. |
| PD-1020 | A Portworx install where the backing drives are provisioned by Portworx can fail with the following fingerprint:<br>``Failed to format [-f --nodiscard /dev/<device-name>]``. The issue can happen when the corresponding device is not attached completely on the node and the format cannot detect it.<br/><br/>**Workaround:** Restart Portworx on the node. To restart Portworx, either label the corresponding Kubernetes node with label `px/service=restart` or if you're already on the node, `systemctl restart portworx`. |
| PD-1017 | The application that uses sharedv4 service encrypted volume may have some pods stuck in the `ContainerCreating` state after there is a sharedv4 service failover and then failback. The failover happens when the Portworx service is restarted on the node where the volume is attached. The failback happens when the Portworx service is subsequently stopped on the node where the volume attachment was moved during failover.<br/><br/>**Workaround:** Restart the Portworx service on the node where the volume was attached before the failover. Use a `normal` failover strategy for the sharedv4 service volumes. The default is `aggressive`. |
| PD-1023 | Failures such as iSCSI disconnects, HBA reset, etc. on Portworx pool drives can sometimes cause the pool to enter an error state and remain in that state if there’s an outstanding operation in the kernel. <br/><br/>**Workaround:** To recover from this, reboot the node. |
| PWX-21956 | Pool expansions by resizing volumes will fail if some paths to the FlashArray are down. Pool expansions may fail in reduced connectivity scenarios, such as Purity upgrades, data network issues, or others.<br/><br/>**Workaround:** Restore all paths to the array before attempting the resize again. |
| PWX-21276 | If some valid and some invalid FlashArray or FlashBlade endpoints are provided, Portworx will fail to start. If invalid credentials or endpoints are entered (or an API token expires), Portworx will fail to start.<br/><br/>**Workaround:** Correct the credentials in the secret and restart Portworx. |
| PD-1024 | When using Portworx with FlashBlade (FB), users store login credentials, such as FA/FB IPs and access tokens, in the `px-pure-secret` Kubernetes secret. In the event that an access token expires or is otherwise invalidated, Portworx automatically provisions workloads onto the next accessible FB to avoid interruptions. <br/><br/>As a result, users may not be alerted when FlashBlades become inaccessible, and workloads can concentrate on the remaining FlashBlades, impacting performance. <br/><br/>**Workaround:** To avoid this issue, ensure the credentials stored in `px-pure-secret` are valid. If you find invalid credentials, correct them and restart Portworx to restore full use.|


## 2.8.1.6

May 20, 2022

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-23997 | A kernel panic occurs if any application tools try to perform a grep operation or user file system level commands on the `pxd-control` device. <br/><br/>**User Impact:** The affected node will experience a kernel panic due to the Portworx kernel module being unable to handle the filesystem user level commands.<br/><br/>**Resolution:** Portworx now handles these kinds of commands on `pxd-control` devices by denying access, preventing kernel panic. |

## 2.8.1.5

Mar 1, 2022

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-22968 | In vSphere environments, unused virtual disks were left lingering around. <br/><br/>**User impact:** Users may have seen multiple KVDB disks lingering around on worker nodes without getting cleaned up. <br/><br/>**Resolution:** Portworx now detects these lingering disks and deletes them. |

## 2.8.1.4

Feb 15, 2022

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-21514 | In vSphere environments, Portworx sometimes failed to remove KVDB drives. <br/><br/>**User impact:** Users saw an additional KVDB drive when they listed all available drives in the Portworx cluster. |

## 2.8.1.3

Jan 24, 2022

### Notes

Portworx now includes kernel module support for `4.15.0-163-generic`.

## 2.8.1.2

Nov 2, 2021

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-21485 | The NFS `rpc-statd` service sometimes failed to start on Portworx nodes, preventing sharedv4 volumes from mounting. <br/><br/>**User Impact:** Application pods using sharedv4 volumes sometimes became stuck in the `ContainerCreating` state, with volume mount operations failing with the error: `mount.nfs: rpc.statd is not running but is required for remote locking.mount.nfs: Either use '-o nolock' to keep locks local, or start statd.\nmount.nfs: an incorrect mount option was specified`<br/><br/>**Resolution:** Portworx now detects when `rpc-statd` is either not running or is in an inconsistent state, and restarts it to ensure that sharedv4/NFS mounts proceed. |

## 2.8.1.1

Oct 13, 2021

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-21506 | One of the Internal folders used for mounting the legacy shared (FUSE) volumes was always created, even if shared (FUSE) volumes were already present on the system. <br/><br/>**User impact:** Mount paths used by shared (FUSE) volumes had wider permissions than desired.<br/><br/>**Resolution:** This change prevents the internal mount path, specifically `/opt/pwx/oci/rootfs/pxmounts`, from being created when there are no shared volumes being used. |

## 2.8.1

Sept 20, 2021

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-21172 | In a Metro  DR configuration, there can be multiple cluster domains within the same cluster. When a sharedv4 volume is created, its replicas are placed across these cluster domains. <br/><br/>If an app requests a sharedv4 volume, then there is no guarantee which replica node will act as the sharedv4 NFS server.<br/><br/>This improvement ensures that whenever a sharedv4 application is started in any of the domains, the volume is attached to a node in the same domain where the application is running, guaranteeing minimum latency. | 

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-21227 | While trying to run IOs on multiple volumes with an overlapped overwrite pattern, an `sm abort` error sometimes occurred on one of the nodes. <br/><br/>**User impact:** This error caused Portworx to restart. <br/><br/>**Resolution:** This rare deadlock during resync no longer occurs. |
| PWX-21002 | A deadlock in the `NFSWatchdog` path caused lock contention and an inspect command to hang. <br/><br/>**User impact:** Users experienced a mounting issue for the affected volume and saw the pod stuck in the `ContainerCreating` state. <br/><br/>**Resolution:** This deadlock has been eliminated. |
| PWX-21224 | Setting cloudsnap threads to 4 or less resulted in cloudsnap backups in hung state. <br/><br/> **User Impact:** With cloudsnap threads (Cloudsnap maximum threads field in cluster options) set to 4 or below and doing more than 10 cloudsnaps at the same time, cloudsnaps would be in hung/stuck state without making any progress. <br/><br/>**Resolution:** Incorrect check for thread count resulted in deadlock causing the above scenario, which has been addressed in this release. |
| PWX-20949 | Issue" Cloudsnap restore may fail with error message "Restore failed due to stall". This happens if restore incorrectly thinks that the node is in maintenance mode. <br/><br/>**User Impact:** These restores may never complete and user may with need to restart the node where the restore is stalled or reissue the restore command<br/><br/>**Resolution:** This change fixes the issue where cloudsnap does not interpret node status. An upper layer module handles it preventing the issue.| 
| PWX-19533 | Fixed an issue where a node may accumulate over-writes for a volume causing the `px-storage` process to restart.<br/><br/>**User impact:** None |
| PWX-21163 | Prometheus was unable to access the Portworx internal etcd on a multi network interface setup, hence etcd alerts are not seen.<br/><br/>**User impact:** Prometheus alerts and monitoring did not work properly for the alerts related to internal etcd on a multi-network interface setup.<br/><br/>**Resolution:** Portworx is now allowed internal etcd access from all network devices, giving Prometheus access to scrape alerts. | 
| PWX-21197 | There was a regression in using `limit_drives_per_pool` in 2.8.0 <br/><br/>**User impact:** `limit_drives_per_pool` is a runtime option to control the number of drives in a pool. The last pool could have more drives than the limit if the drive count was not an exact multiple of the limit, and if creating another pool would have resulted in too few drives within it. Internally, a new pool is created if the drive count is at least 50% of the limit available in the last pool.<br/><br/>**Resolution:** These limits are now honored more strictly when a drive is added from maintenance mode. Any drive add operation will fall into a pool only if the drive count is within the limit. If not, a new pool will be formed. | 

## 2.8.0

July 30, 2021

### New features

Portworx by Pure Storage is proud to introduce the following new features:

* Early access support for [Pure FlashBlade as a Direct Access filesystem](/operations/operate-kubernetes/storage-operations/create-pvcs/pure-flashblade). With this feature, Portworx directly provisions FlashBlade NFS filesystems, maps them to a user PVC, and mounts them to pods. Reach out to your account team to enable this feature. 
* Early access support for [Pure FlashArray cloud drives](/cloud-references/auto-disk-provisioning/pure-flash-array/). Use FlashArrays as a cloud storage provider. Reach out to your account team to enable this feature. 
* [Snapshot optimization using extent metadata](/operations/operate-kubernetes/storage-operations/create-snapshots/snapshot-methods): Reduce the amount of data sent to your cloud storage provider when taking cloud snapshots. 
* [SkinnySnaps](/operations/operate-kubernetes/storage-operations/create-snapshots/skinnysnaps): improve the performance of your storage pools when taking volume snapshots. 
* [Sharedv4 service volumes](/operations/operate-kubernetes/storage-operations/create-pvcs/create-sharedv4-pvcs): improve fault tolerance by associating sharedv4 volumes with a Kubernetes service.
* You can now install [Portworx on Nomad with CSI enabled](/install-portworx/install-with-other/nomad/installation/install-as-a-nomad-job).
* [Install and scale](/cloud-references/auto-disk-provisioning/tanzu/install) a Portworx cluster on VMware Tanzu with CSI.
* [With Pure1 integration](/operations/operate-kubernetes/troubleshooting/collecting-diagnostics), Portworx can now automatically upload its diags to Pure Storage's call home service called **Pure1**.

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-20720 | Portworx now supports SharedV4 volumes on VMware Photon hosts |
| PWX-20131 | You can now resize disk pools with disks of size 32TiB on Azure for up to 32TiB. |
| PWX-20060 | The Portworx spec generator now creates the GA/v1 API version of the CSI VolumeSnapshot CRDs. |
| PWX-18845 | Portworx now supports Amazon General Purpose SSD volumes (gp3). | 
| PWX-10281 | The Portworx CSI driver now supports Raw Block volumes for RWO PVCs. | 
| PWX-20102 | Licensing improvement: Autopilot licenses are now automatically included with all Portworx Enterprise licenses, including floating licenses.|
| PWX-19553 | Alerts now bypass the API queue. As a result, `pxctl` will still show alerts even when the API queue is full. | 
| PWX-19496 | Kubernetes PVCs will now get created by the CopyOnWrite on demand setting by default. |
| PWX-19320 | Portworx CSI Driver volumes will now be rounded up to the nearest GiB to match the Portworx in-tree volume plugin. This change only occurs for new volumes and volume size updates. | 
| PWX-20803 | Added photon support for pool caching. Portworx will try to install the required packages if enabled. If they fail, the Portworx installation will fail. Pool caching requires the following available packages: <ul><li>device-mapper</li><li>mdadm</li><li>lvm2</li><li>thin-provisioning-tools</li></ul> |
| PWX-20527 | The maximum number of cloud drives per node (not per pool) has increased from 12 to 32. Note that specific cloud providers may impose their own limits (remaining at 12), and that there are still limits per pool that may come into effect sooner. | @Adam Krpan
| PWX-20423 | Sharedv4 Export Options Improvements: The storage class option `export_options` can now take any NFS export option as a comma separated list of strings. Portworx will apply those export options on the node where the volume is attached and exported over NFS. |
| PWX-20204 | For cloud drive, the requirement to specify `skip_deprecation` has been removed and users can now use the `pxctl sv drive add` command without that. |
| PWX-18529 | Portworx will now report back home in trial license installations | 


### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-20780 | Pods using encrypted sharedv4 volumes sometimes got stuck in the terminating state. <br/><br/>**User impact:** If a node hosting a replica of a sharedv4 encrypted volume (server node) was rebooted, the application pods accessing that volume sometimes got stuck in the terminating state. <br/><br/>**Resolution:** Portworx now detects when a `sharedv4` encrypted volume server node restarts, and will automatically restart application pods using that volume to recover them to a functional state. |
| PWX-20417 | On Kubernetes-v1.20.1 on the containerd-v1.4.4 container runtime,the Portworx installer used an invalid cgroups-path. <br/><br/> **User impact:** Portworx failed to install <br/><br/> **Resolution:** Portworx now properly installs on this Kubernetes version and runtime. |
| PWX-20789 | A pool expand status message was incorrect. <br/><br/>**User impact:** The status message in the pool expand operation that appears in the `pxctl service pool show` command output was inaccurate when the operation type was `add-disk`. Users saw the incorrect size amount by which the pool was being expanded by, but the operation functioned properly. <br/><br/>**Resolution:** The status message now correctly states the size of the the pool that is expanding.|
| PWX-20327 |  When running on cloud at scale (more than 200 nodes), if there are certain nodes flapping or restarting in a loop, it would cause healthy Portworx nodes to slow down processing kvdb watch updates. <br/><br/>**User impact:** Certain volume operations (for example, create or attach) take a long time or fail. | 
| PWX-20085 | Portworx failed to install when both `-metadata` and `-kvdbDevice` options were passed. <br/><br/>**User impact:** Users saw their installations fail if they provided both options. <br/><br/> **Resolution:**  If users provide both options, Portworx installation will no longer fail. It now defaults to using the `-metadata` device for KVDB to maintain backward compatibility. |
| PWX-19771 | Communication between nodes could block forever. <br/><br/>**User impact:** Users sometimes saw volume access or management pend forever without timing out. <br/><br/>**Resolution:** All communications in the Portworx cluster now timeout. A new `pxctl cluster options` allows you to configure a default RPC timeout. | 
| PWX-20733 | If a custom container registry was specified and the cluster was using containerd, the oci-monitor attempted to pull the Portworx Enterprise image from the wrong registry. <br/><br/> **User impact:** Users saw installation fail. <br/><br/>**Resolution:** oci-monitor now pulls the Portworx Enterprise image from the correct custom registry. | <!-- @Zoran Rajic -->
| PWX-20614 | When multiple oci-monitor container images are present (using the same image-hash but a different name), Kubernetes sometimes started the wrong container image. <br/><br/>**User impact:** Users potentially saw the wrong Portworx Enterprise image to be loaded on the nodes. <br/><br/>**Resolution:** The oci-monitor now consults multiple configuration entries when deciding which px-enterprise image needs to be loaded." |
| PWX-20456 | Fixes the installation problem with containerd-v1.4.6 container runtime, running in "systemd cgroups" mode.<br/><br/>**User impact:** If one is running a Kubernetes cluster configured with containerd-v1.4.6 (or higher) container runtime with systemd cgroups driver, Portworx service would fail to start.<br/><br/>**Resolution:** Portworx startup issues are now resolved when using this container runtime/configuration.| 
| PWX-20423 | The `security_label` export option was removed after a node reboot. <br/><br/>**User Impact:** When using `sharedv4` with **SELinux** enabled, an app using `sharedv4` volumes sometimes saw permission issues if the node where the volume was attached was rebooted. <br/><br/>**Resolution:** Improvements to the `sharedv4` volumes resolved this issue. |
| PWX-20319 | Fixes an issue with Portworx service restarts hanging indefinitely, when DBus service is not responding. <br/><br/>**User impact:** On host-systems that have a non-responsive DBus service, Portworx startup used to hang indefinitely. <br/><br/> **Resolution:** Portworx startup no longer hangs if it cannot connect to the DBus service. |
| PWX-20236 | In some scenarios, volume unmount may fail due to EIO errors on mount path, which could be due to prolonged downtime on the volume.  <br/><br/>**User Impact:** Pods may fail to terminate and reboot. <br/><br/>**Resolution:** Continue to unmount the volume even when readlink fails with EIO on mount path. This allows pods to continue with remount of the volume.  | 
| PWX-20187 | When passing a Kubernetes secret for etcd username/password using environment variables, they were taken and used "as is", rather than being "expanded" and replaced with the actual values from Kubernetes secret. <br/><br/>**User impact:** When specifying the etcd username/password, the environment variables (e.g. populated by Kubernetes secrets) were not being "expanded" before configuring etcd connection. <br/><br/>**Resolution:** You can now specify the etcd username/password via the environment variables. |
| PWX-20092 | Queued backups may fail if the volume replica is reduced in such a way that the replica on the node assigned for queued backup gets removed. <br/><br/>**User Impact:** Queued backups failed. <br/><br/>**Resolution:** Users can wait for queued backups to complete before running the ha-reduce command, or re-issue the backup command once the HA reduce is complete. |
| PWX-19805 | Portworx couldn't unmask the `rpcbind` service. <br/><br/>**User impact:** Portworx could not properly integrate with NFS services, if NFS services were masked on the host. As a consequence, sharedV4 volumes could not be served from that host. <br/><br/>**Resolution:** The Portworx service startup/restart now checks for masked NFS services, and automatically unmasks them. | 
| PWX-19802 | Migrations were failing with error - "Too many cloudsnap requests please try again" <br/><br/>**User Impact:** If a cluster migration was triggered which migrated more than 200 volumes at once, the migration would fail since Portworx would rate limit the cloudsnap requests. <br/><br/>**Resolution:** Cluster Migration will not fail if the internal cloud snap requests are rate limited. Portworx will gracefully handle those "busy" errors and retry the operation until it succeeds. |
| PWX-19568 | When an NVMe partition is specified as a storage device, the device state is shown as "offline". <br/><br/>**User impact:** Users sometimes saw their partitioned NVMe drive state as "offline". <br/><br/>**Resolution:** NVMe partitions given as storage devices now correctly show as "online". | 
| PWX-19250 | Talisman px-wipe does not support etcd with username and password <br/><br/>**User impact:** Using automated cluster-wipe procedure would not purge the cluster's data from the key-value database, when an external username/password -enabled EtcD was configured as cluster's KVDB. <br/><br/>**Resolution:** The cluster-wipe procedure should now purge the data also from username/password -enabled EtcD." | 
| PWX-19209 | OCI-Mon/containerd occasional install/upgrade glitches <br/><br/>**User impact:** During Portworx upgrades on containerd container-runtimes, there was a race-condition with the cleanup of install-container, which could block the upgrade until the node got rebooted. <br/><br/>**Resolution:** The cleanup procedure was improved, so the race-condition no longer occurs" |
| PWX-18704 | Drive add not respecting limit_drive_per_pool <br/><br/>**User impact:** When a runtime option `limit_drive_per_pool` was set to 2, a pool delete/initialize/drive add operation sometimes resulted in pools of more than 2 drives.<br/><br/>**Resolution:** The `px-runc` install-time configuration `limit_drives_per_pool` is now honoured during drive.  | 
| PWX-18370 | A tracker file from previous Portworx version was not getting deleted during wipe causing sharedv4 volume mount issues. <br/><br/>**User impact:** If you deleted a Portworx version and reinstalled a new version, the wipe process did not remove a tracker file used for sharedv4 volumes. This will fail to mount new sharedv4 volumes after reinstallation. <br/><br/>**Resolution:** The tracker file is now removed.|
| PWX-20485 | In versions earlier than Portworx 2.7.0, pxctl volume check on ext4 formatted secure pxd volume returns the "Background Volume Service not supported on Encrypted volumes" error. <br/><br/>**User Impact:** Users couldn't use `pxctl` to do volume checks on ext4 formatted secure pxd volumes. A workaround was fsck directly from within the Portworx container. <br/><br/>**Resolution:**  Portworx now supports volume check on ext4 formatted secure pxd volume using pxctl. |
| PWX-20303 | Fixed an issue where KVDB updates were not pulled in from other nodes when this node is unable to get the updates from KVDB. | 
| PWX-20398 | Fixed an issue where request processing started before the px-storage process was initialized causing it to restart. <br/><br/>**User impact:** These restarts may have created core files, users may have seen the process restart.|
| PWX-20690 | The px-storage process incorrectly finished processing internal timestamps causing it to restart. <br/><br/>**User impact:** These restarts may have created core files, users may have seen the process restart.  |
| PWX-19005 | Cloudsnaps may be stuck irrespective of whether you issued stop on the cloudsnap taskID. This sometimes happened when the local snapshot was deleted while cloudsnap was still active. In these conditions, snap detach operations failed and caused cloudsnaps to get stuck. <br/><br/>**User Impact:** Users saw stuck cloudsnaps, which could only be fixed by restarting the node where the stuck cloudsnap was active. <br/><br/>**Resolution:** Detach failures are not retried forever, minimizing this scenario. |
| PWX-20708 | Cloudsnap size was not tracked correctly for incremental cloudsnaps in cloudsnap metadata. <br/><br/>**User Impact:** Users may not see correct cloudsnap size because of this issue. <br/><br/>**Resolution:** Now the incremental cloudsnap size is tracked correctly. |
| PWX-20364 | An incorrect check searched all nodes in ReplicaSets to be online. <br/><br/>**User impact:** Cloudsnaps may fail on restart if some of the replica nodes are online. <br/><br/>**Resolution:** Removed the check for online nodes. | 
| PWX-20237 | Cloudsnap operations did not choose the node where the previous cloudsnap was executed, specifically for reducing the HA level volume. <br/><br/> **User impact:** Some cloudsnaps could be shown as full, even though it could have been incremental. <br/><br/>**Resolution:** If the volume replicas differ between previous cloudsnap and current cloudsnap, Portworx now chooses the same replica node where the previous backup ran. |


### Known issues (Errata)

Portworx is aware of the following issues, check future release notes for fixes on these issues:

|**Issue Number**|**Issue Description**|
|----|----|
| PD-896 | With 2.8.0, the `--sharedv4_service_type` option was added to the `pxctl volume create` and `pxctl volume update` commands. This new option is not applicable to non-Kubernetes environments, such as `Nomad`. This option works only in the Kubernetes environment. |
| PWX-20666 | The issue occurs during the first boot of a newly created cluster, and if the node contains Ubuntu 20.04/Fedora CoreOS 33.20210301.3.1 as the host and you use the `-j auto` option. Any Linux distribution using `systemd` version 245 and above seems to have this problem. Ubuntu 18.04 containers use version 237 of `systemd`. <br/><br/>**User impact** Portworx takes additional time to come up.<br></br>**Recommendation:** Do not use the `-j auto` option. If you use the option, then consider installing parted command on the host. If that is not possible, wait for Portworx to reach stability. You must only wait for the first boot after new cluster creation.|
| PWX-20836 | When using Portworx cloud drives and using a node selector in the StorageCluster CRD specification, a node may come up with a drive configuration that is different from what is specified in the StorageCluster CRD under the node selector. <br/><br/> This occurs when one Portworx node creates a drive based on the node selector configuration. If that drive is not currently being used by that node, another node that is trying to initialize may pick up this available drive (even if that newly created drive does not match with the user specified configuration for that node).| 
| OPERATOR-410 | A workaround for deploying Operator on OCP (IBM) with nodes labeled as worker/master: Remove the nodeAffinity `node-role.kubernetes.io/master` from the live StorageCluster spec to deploy OCI pods on control plane nodes. | 
| PD-891 | Currently Photon OS versions 3.0 and 4.0 do not allow a user to install `mdadm` using default `yum` and `tdnf` repositories. As this package is a must for px-cache deployment Pure Storage does not support caching on Photon OS on version 2.8.0 | 
| PWX-20839 | Portworx 2.8.0 does not support the Photon distro for px-cache. |
| PWX-20586 | When creating a burst of PVCs (a large number of PVCs within few seconds) with the Portworx CSI driver or a separate Portworx PVC controller, the Portworx volume creations can get stuck in a pending state. Under this condition, `pxctl volume list` will show these volumes as “down - attached”. The volume creates will eventually converge and complete, but this can take more than 1 hour.<br/><br/>**Workaround:** Stagger PVC creations in smaller batches if using the CSI driver or the Portworx PVC controller.|
| PWX-13190 | The `pxctl credentials delete <uid>` command is currently not supported for Kubernetes secret provider. To delete the credentials use `kubectl delete secret <> -n <namespace>`, where `<namespace>` is where Portworx is installed. |
| PWX-9487 | In a metro DR setup where Portworx spans across 2 data centers, a node can be started with an argument `--cluster_domain` and value set to `witness`. This node will act as a witness node between the two data centers and it will also contribute to quorum even if it is started as storageless node. | 
| PWX-20766 | The Portworx CSI Driver on Openshift 3.11 may have issues starting up when a node is rebooted. Customers are advised to upgrade to Openshift 4.0+ or Kubernetes 1.13.12+.|
| PWX-20766 |  The Portworx CSI Driver will not be enabled by default in the PX-Installer for Kubernetes 1.12 and earlier unless a flag `csiAlpha=true` is provided. |
| PWX-20423 | Portworx uses a set of default export options in storage classes which cannot currently be overridden. The `export_options` parameter only allows extending the current default options. | 
| PD-915 | If a storage node is removed or replaced from the cluster and that node was the NFS server for a sharedv4 volume, client application pods running on other nodes can get stuck in pod Terminating state when deleted.<br/><br/>**User impact:** Application pods using sharedv4 volumes remotely will get stuck in Terminating state.<br/><br/>**Recommendation:** <ul><li>Find the node where the pod is stuck in terminating state</li><li>Find pod UUID: <code>kubectl get pods -n <namespace> <pod> -o yaml \| grep uid \| tail -n1</code></li><li>Run <code>mount \| grep \<pod_uuid\> \| grep \<vol-name\></code></li><li>Unmount the path: <code>umount -f -l \<mounted_path\></code></li></ul> |
| PD-914 | When a pod that is using a sharedv4 *service* volume, is scheduled on the same node where the volume is attached, Portworx sets up the pod to access the volume locally via a bind mount. If the pod is scheduled on a different node, the pod uses an NFS mount to access the volume remotely. If there is a sharedv4 service failover, the volume gets attached to a different node. After the failover, pods that were accessing the volume remotely over NFS, continue to have an access to the volume. But the pods that were accessing the volume locally via bind-mount, lose access to the volume even after Portworx is ready on the node since the volume is no longer attached to that node. Such pods need to be deleted and recreated so that they start accessing the volume remotely over NFS. If the stork is enabled in the Kuberenetes cluster, it automatically deletes such pods. But sometimes a manual intervention may be required if the stork is either not installed or fails to restart such pods. <br/><br/>**Recommendation:** Enable stork to reduce the likelihood of running into this issue. If you do run into this issue, use the command `kubectl pod delete -n <namespae> <pod>` to delete the pods which cannot access sharedv4 service volume anymore because the sharedv4 service failed over to another node. <br/><br/>This problem does not apply to the pods that are using "sharedv4" volumes without the `service` feature. <br/><br/>This problem does not apply to the pods that are accessing "sharedv4 service" volumes remotely i.e. from a node other than the one where volume is attached. |
| PD-926 | For information about Sharedv4 service known issues, see the notes in the [Provision a Sharedv4 Volume](/operations/operate-kubernetes/storage-operations/create-pvcs/create-sharedv4-pvcs) section of the documentation.  |

## 2.7.4

August 27, 2021

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-21057 | Pods failed to come up with restored PVCs that were encrypted with vault namespace secrets. <br/><br/>**User impact:** Pods using PVC that is cloned from snapshot, which is encrypted using vault namespace secrets, will remain in container creating. <br/><br/>**Resolution:** Portworx now fixed this issue to copy over the required values for encrypted volumes for cloned PVC. Pods will not remain in container creating for this issue. |
| PWX-21139 | In a DR setup, during the failover and failback of an encrypted volume, some labels that Portworx used for encrypting the volumes got removed. <br/><br/>**User impact:** Failback or restore of encrypted volumes using per-volume secrets would fail as restore of the volume on the source cluster would fail. <br/><br/>**Resolution:** The cloud backups and restores done as a part of failover and failback of encrypted volumes ensure that the encryption related labels are not removed. |

## 2.7.3

July 15, 2021

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-20323 | Portworx now tries to reconnect to the KVDB at least 3 times before restarting the Portworx process. |
| PWX-19994 | Added two new runtime options: <br/><br/>`quorum_timeout_in_seconds`: Sets the maximum time for which nodes will wait in seconds to reach quorum. After this timeout, Portworx will restart.<br/><br/>`kv_snap_lock_duration_in_mins`: Sets the maximum timeout for which Portworx will wait for a KVDB snapshot operation to complete. After this timeout, Portworx will panic and restart if the snapshot does not complete.| 

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-20641 | RedHat OpenShift 4.7.16 write locks partitions Portworx uses during upgrades. Portworx versions 2.5.x through 2.7.2.1 generated chattr-protected immutable /etc/pwx/.private.json files. Lastly, OpenShift 4.7 started protecting root-partition (using read-only mountpoints).<br/><br/>**User impact:** These files were sometimes collected into OpenShift's historical snapshots and interfered with OpenShift upgrades.<br/><br/>**Resolution:** the `px-runc` statup service now scans and fixes any immutable .private.json files found in CoreOS file-system snapshots and OpenShift snapshots. This works for both read-only and read-write partitions. | 
| PWX-15391 | When using an internal KVDB, Portworx encountered an error if one of the KVDB nodes went down. <br/><br/>**User impact:** Users saw impacted filesystems enter read-only mode. <br/><br/>**Resolution:** The run-flat feature keeps the Portworx volumes online, even if the KVDB is down. New create/attach/mount operations are not allowed, but existing volumes don't see an I/O interruption as long as all the replicas of the volume are online. Note the following implications for the external and internal KVDB: <br/><br/>**External KVDB:** as long as all the Portworx nodes continue to stay up, all the volume replicas should stay online and with no I/O interruption. However, if any node goes down, volumes with a replica on the down nodes will see an outage. <br/><br/>**Internal KVDB:** if the KVDB is down, it almost certainly implies that at least 2 Portworx nodes are down. Any volumes with a replica on the down nodes will see an I/O interruption. |
| PWX-20075 | CSI VolumeSnapshotContent objects incorrectly displayed a restore size of 0.<br/><br/>**User impact:** External backup systems that depend on the CSI VolumeSnapshotContent restore resize sometimes failed. <br/><br/>**Resolution:** The Portworx CSI driver now correctly adds the restore size to new CSI volume snapshots, and snapshot contents will have the correct RestoreSize. |
| PWX-19518 | Overwriting a cluster wide secret when using Vault Namespaces failed with the `NotFound` error. <br/><br/>**User impact:** Users were unable to use Vault as their secret management store. <br/><br/>**Resolution:** The issue has been fixed and Portworx now uses the correct vault namespace while resetting the cluster wide secret. |
| PWX-20149 | Portworx encrypted volume creation failed due to lock expiration.<br/><br/>**User impact:** In certain scenarios, Portworx encrypted volume creation took longer than expected and eventually timed out. <br/><br/>**Resolution:** Portworx no longer times-out when creating encrypted devices. |
| PWX-20519 | On vSphere environments experiencing high I/O latency, Portworx cluster installation failed while setting-up the internal KVDB. <br/><br/>**User impact:** Users saw the internal KVDB fail to initialize the disks within the allocated time.<br/><br/>**Resolution:** Portworx now initializes a "thin" disk, rather than a "zeroedThick" disk by default; this option can be overridden. |

## 2.7.2.1

June 25, 2021

### Notes

* Portworx 2.7.2.1 once again supports installations on Ubuntu 16.04 with the 4.15.0-142-generic and 4.15.0-144-generic kernels. See the list of [supported kernels](/reference/supported-kernels) for information.

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-20594 | Portworx erroneously allowed the creation of large replication sets, causing the `px-storage` process to create a core file.<br/><br/>**User impact:** Users required help from support to fix the volume definition.<br/><br/>**Resolution:** Portworx no longer allows the creation of replication sets larger than 3. |
| PWX-20629 | I/O did not progress for Portworx volumes and mount points until the `px-storage` process was restarted. <br/><br/>**User impact:** I/O may not have progressed and users may have had to restart the `px-storage` process.<br/><br/>**Resolution:** Portworx no longer requires a restart of the `px-storage` process. |
| PWX-20619 | I/O to the sharedv4 volumes was blocked while the NFS server reloaded the exports. When there were a large number of sharedv4 volumes being exported from the node, I/O was blocked for a prolonged period of time. <br/><br/>**User impact:** Apps using the `df` command saw it take a long time to complete, causing pods to reset unnecessarily when `df` was used as a health-check. Removing one of the export options fixed this issue.<br/><br/>**Resolution:** The `df` command will no longer slow down under these circumstances. |
| PWX-20640 | When uploading diagnostics, Portworx made its configuration file immutable. <br/><br/>**User impact:** When made immutable, configuration files may interfere with Opershift upgrades.<br/><br/>**Resolution:** Portworx no longer makes its configuration file immutable when uploading diagnostics. | 



## 2.7.2

May 18, 2021


### Notes

* Portworx 2.7.2 no longer supports installations on Ubuntu 16.04 with the 4.15.0-142-generic kernel. Upgrade to Ubuntu 18.04. See the list of [supported kernels](/reference/supported-kernels) for information.

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-20106 | Portworx now supports various kernels hosted on IBM Cloud in air-gapped environments. Specifically, Portworx supports the 4.15.0-142-generic kernel on Ubuntu 18.04. |
| PWX-20072 | Users can now install Portworx from the IBM catalog onto a private cluster and enable the integrated license and billing feature for these clusters. |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-14559 | The Read and write throughput values in Grafana and Prometheus were erroneously transposed. <br/><br/>**User impact:** Users saw read throughput values where they expected to see write throughput values, and vice versa. <br/><br/>**Resolution:** These values now reflect the correct metric in Grafana and Prometheus. |

## 2.7.1

April 29, 2021

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-18530 | Storage pool caching now supports caching on SSD pools, which will be cached with NVME drives if they're available. | 

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-19368 | If Floating licenses were used on a cluster, wiping a node or the cluster didn't return the license leases back to the license server. <br/><br/>**User impact:** Users had to wait for the license leases to expire in order to reuse them. <br/><br/>**Resolution:** Wiping a node or cluster now correctly releases the license leases back to the license server. |
|  PWX-19200 | Portworx was unable to attach AWS cloud drives when running on AWS Outpost. <br/><br/>**User Impact:** When running on AWS Outpost, Portworx failed to attach the backing drives, causing the cluster initialization to fail. <br/><br/>**Resolution:** Portworx now includes the Outpost ARN in the EBS volume creation, which allows the volume to be attached in the instance | 
|  PWX-19167 | In very rare cases, the `px-storage` process sometimes aborted and restarted due to a race condition when releasing resources.<br/><br/>**User impact:** Users experienced no interruptions, but may have seen Portworx restart.<br/><br/>**Resolution:** This race condition no longer occurs. |
| PWX-19022 | Pods sometimes failed to mount a volume and may not have started if that volume was first attached for background work and then later attached for mounting purposes. <br/><br/>**User impact:** Users saw their pods fail to start. To correct this, they had to detach and reattach the volume, usually by stopping and restarting the affected application. <br/><br/>**Resolution:** Pods no longer fail to mount volumes under these circumstances.|
| PWX-18983 | An issue with security and non-CSI deployments during volume deletion caused Portworx to return incorrect information when it detected an error during a request to inspect a volume. <br/><br/>**User impact:** Users saw incorrect information when inspecting a deleted volume. <br/><br/>**Resolution:** Portworx now displays the correct information when inspecting a volume under these circumstances. |
|  PWX-18957 | After recovering an offline cluster from a KVDB backup file, that cluster's license entered an invalid state. This was caused by the ClusterUUID in the restored KVDB having extra quotes. <br/><br/>**User impact:** Users attempting to recover clusters in this manner saw their licenses fail to restore correctly. <br/><br/>**Resolution:** During the recovery process, Portworx now ensures that no extra quotes are added to the ClusterUUID once the recovery is done. |
|  PWX-18641 | Portworx displayed an incorrect alert for snapshots when the parent volume's HA level was decreased. <br/><br/>**User impact:** Users may have seen this incorrect alert. <br/><br/>**Resolution:** Portworx will no longer attempt to run HA level reduce operations on snapshots which have an HA level of 1 (which fails and triggers incorrect alert) when the HA level is reduced on the snapshot's parent volume. |
| PWX-18447 | Portworx enabled the sharedv4/NFS watchdog even if the `--disable-sharedv4` flag was set. <br/><br/>**User impact:** Despite disabling the sharedv4 volume feature, users may have seen errors about NFS/sharedv4 being unhealthy. <br/><br/>**Resolution:** Portworx no longer enables the NFS watchdog if sharedv4/NFS is disabled. |
| PWX-17697 | Users couldn't remove storage pool labels. <br/><br/>**User impact:** When users attempted to remove storage pool labels, they saw the command return `Pool properties updated`, but Portworx didn't remove the label.<br/><br/>**Resolution:** Storage pools now have the same behavior as volumes. You can now remove labels by passing `--labels <key=>` without a value to remove the previously added label. |
| PWX-7505 | Unsecured nodes could be added to a secured cluster, specifically if those nodes are part of a different Kubernetes cluster with a different configuration manifest. <br/><br/>**User impact:** This allowed any of the unsecured nodes to join a secured cluster. As a result, the API endpoints for the unsecured nodes would be unsecured and allow anyone to execute any pxctl or RPC request. <br/><br/>**Resolution:** Portworx can now be configured using the `PORTWORX_FEATUREGATE_CHECK_NODE_SECURITY` feature gate to prevent unsecured nodes from joining a cluster if at least one node is secured. |
| PWX-19503 | The `px-storage` process initialization got stuck if the "num_cpu_threads" or "num_io_threads" rt_opts value did not equal the "num_threads" rt_opt value.<br/><br/>**User impact:** Portworx didn't come up, and users needed to remove the "num_threads" rt_opts for the `px-storage` process to finish initialization.<br/><br/>**Resolution:** Portworx initialization no longer gets stuck. |
| PWX-19383 | A Kubernetes RBAC issue with the CSI resizer installation caused CSI PVC resizing to fail.<br/><br/>**User impact:** Users saw CSI PVC resize failures. <br/><br/>**Resolution:** The Portworx spec generator now correctly adds the necessary RBAC for the CSI Resizer to function properly. |
| PWX-19173 | CSI VolumeSnapshotContent objects incorrectly displayed a restore size of 0.<br/><br/>**User impact:** External backup systems that depend on the CSI VolumeSnapshotContent restore resize sometimes failed. <br/><br/>**Resolution:** The Portworx CSI driver now correctly adds the restore size to a VolumeSnapshotContent object. |
| PWX-18640 | The Portworx alert VolumeHAUpdateFailure has been updated to VolumeHAUpdateNotify for cases where the update is not failing. <br/><br/>**User impact:** Users saw misleading VolumeHAUpdateFailure alerts when an update succeeded. <br/><br/>**Resolution:** Portworx alerting system sends the correct alarm event for this case. |
| PWX-19277 | Cloudsnaps sometimes failed to attach the internal snap for an aggregated volume if the node containing the aggregated replica was down. <br/><br/>**User impact:** While the cloudsnap operation was marked as failed, the error description did not display the correct error message. <br/><br/>**Solution:** Cloudsnaps no longer fail to attach, and error messages now correctly indicate that the node is down. |
| PWX-19797 | With 2.7.0, cloudsnap imposed restrictions on active cloudsnap commands being processed. <br/><br/> **User impact:** Async DR sometimes failed for some volumes. <br/><br/> **Solution:** 2.7.1 increases the number of commands being processed to a much higher value, thereby avoiding async DR failures. | 

## 2.7.0

March 23, 2021

### New features

* Announcing the [`auto` IO profile](/concepts/io-profiles#the-auto-profile), which applies an IO profile to optimize volume performance based on the workload data patterns it sees.

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-16376 | Multiple KVDBs will now run within the same failure domain if there aren't enough failure domains available to place each KVDB on its own.|
| PWX-11345 | Introducing a new Prometheus metric to track latencies Portworx sees from the KVDB. This metric tracks the total time it takes for a KVDB put operation to result in a corresponding watch update from the KVDB.  |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-18789 | Expanding a pool using the `lazyzeroedthick` disk type in vSphere failed with the error: "found no candidates which have current drive type: lazyzeroedthick". <br/><br/>**User impact:** If a user expanded a pool using the `lazyzeroedthick` disk type in vSphere, it failed with a message like: "could not find a suitable storage distribution candidate due to: found no candidates which have current drive type: lazyzeroedthick". <br/><br/>**Resolution:** This happened because a section for the `lazyzeroedthick` disk type was missing in the storage decision matrix config map that ships with Portworx; this has now been added. |
| PWX-17578 | Updating volumes with `pxctl` commands sometimes reset other values that were previously set by a spec. <br/><br/>**User impact:** If users updated the `queue-depth` value using the `pxctl volume update --queue-depth` command, the `directIO` value for that volume was reset.<br/><br/>**Resolution:** Portworx no longer resets volume-specific fields when other fields are updated using `pxctl` commands. |
| PWX-18724 | When using the runtime option `rt_opts_conf_high` under heavy load, the Portworx storage process sometimes ran out of internal resources and had to restart due to an assertion failure. <br/><br/>**User impact:** Upon restart, the Portworx process  may have gotten stuck in a restart loop, resulting application downtime. <br/><br/>**Resolution:** The resources are now sized correctly when the `rt_opts_conf_high` runtime option is in use. | 
| PWX-18632 |Portworx displayed expiration dates for permanent licenses.<br/><br/>**User impact:** Users saw a distinct expiration date for their permanent licenses. Despite this reporting error, permanent licenses would not actually expire. <br/><br/>**Resolution:** Portworx now correctly reports permanent licenses as never expiring. |
| PWX-18513 | If a volume with an io_profile set to `db_remote` and replication level of `1`  was backed-up using a cloudsnap, attempting to restore that cloudsnap would fail through Stork or in PX-Backup. <br/><br/>**User impact:** Users attempting to restore this kind of cloudsnap without providing an additional parameter to force the replication level to 2 encountered an error.<br/><br/>**Resolution:** The cloudsnap restore operation now resets the io_profile to `sequential` if it finds a volume with  a replication level of `1` and io_profile set to `db_remote`. |
| PWX-18388 | Pool expand operations using the `resize-disk` method failed on vSphere cloud drive setups. <br/><br/>**User impact:** Users with storage pools powered by vShpere cloud drives could not expand them using the `resize-disk` method.<br/><br/>**Resolution:** The `resize-disk` method failed because the `rescan-scsi-bus.sh` script was missing from the Portworx container. This script has been replaced, and users can once again expand vSphere cloud drive storage pools using `resize-disk`. |
| PWX-18365 | Portworx overrode the cluster option for optimized restores if a different runtime option for optimized restores was provided. <br/><br/>**User impact:** Because Portworx prefers cluster options over runtime options as a standard, users may have been confused when this runtime option behaved differently.<br/><br/>**Resolution:** Portworx no longer honors runtime options for optimized restores; you must use [cluster options](/reference/cli/cluster/#enabling-optimized-restores) to enable optimized restores.|
| PWX-18210 | The `px/service` node-label is used to control the state of the Portworx service. However, the `px/service=remove` label did not properly remove Portworx.<br/><br/>**User impact:** When users attempted to remove a node, Portworx became stuck in an uninstall loop on that node.<br/><br/>**Resolution:** The `px/service=remove` label now behaves as it previously did, and now uninstalls Portworx on the node as expected. |
| PWX-17282 | Previously, every Portworx deployment using the Operator included Stork, regardless of whether or not you enabled the Stork component on the [spec generator](https://central.portworx.com/). <br/><br/>**User impact:** Users' deployments always included Stork, even if they did not want to enable it. <br/><br/>**Resolution:** The spec generator now correctly excludes Stork if you don't enable it. |
| PWX-19118 | A resize operation performed while a storage node was in the `StorageDown` state caused the `px-storage` process to restart if the node had replica for the volume being resized. <br/><br/>**User impact:** Users experienced no interruptions, but may have seen Portworx restart. <br/><br/>**Resolution:** the `px-storage` process no longer restarts under these circumstances. |
| PWX-19055 | Portworx clusters did not auto-recover when running on vSphere cloud drives in local mode. <br/><br/>**User Impact:** When users installed Portworx using vSphere cloud drives on local, non-shared datastores and used an internal KVDB, Portworx did not automatically recover if a storage node went down. <br/><br/>**Resolution:** Portworx will no longer incorrectly mark its internal KVDB drives as storage drives, allowing the internal KVDB to recover as expected. |
| PWX-17217 | Portworx failed to exit maintenance mode after a drive add operation was shown as `done`. <br/><br/>**User impact:**  Portworx stays in maintenance mode and users cannot exit it. <br/><br/>**Resolution:** Portworx now properly exits maintenance mode after a drive add operation. |
| PWX-19060 | When Portworx was configured to use email, an entry was printed to the log that contained hashed email credentials. <br/><br/>**User impact:** Hashed, potentially sensitive information may have been written to the logs. <br/><br/>**Resolution:** Portworx no longer prints this hashed information into the log. |
| PWX-19028 | Portworx sometimes hung when evaluating multipart licenses where one of the licenses had expired. <br/><br/>**User impact:** Users saw Portworx hang and had to reset the Portworx node. <br/><br/>**Resolution:** Portworx no longer hangs in this scenario.|

### Notes

* Portworx 2.7.0 is not currently supported on Fedora 33 

### Known issues (Errata)

Portworx is aware of the following issues, check future release notes for fixes on these issues:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-19022 | Attaching/Mounting a volume on the same host where it was attached internally for background work (such as cloudsnaps) fails to create the virtual kernel device. <br/><br/>**User impact:** The pod may fail to mount the volume and may not start. <br/><br/>**Recommendation:** Detach and re-attach the volume to fix this issue. |

## 2.6.5

March 6, 2021

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-18967 | Portworx occasionally locked volume creation for a prolonged period when taking more than one cloudsnap for the same volume.  <br/><br/>**User Impact:** Users experienced longer response times when creating new volumes. <br/><br/>**Resolution:** Cloudsnaps now lock only the volume they're being taken on, and no longer interfere with volume creation. |

## 2.6.4.1

February 18, 2021

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-18625 | In certain corner cases, when a volume being restored on a destination cluster was deleted before a restore completed, async DR or migration sometimes got stuck. <br/><br/>**User Impact:** Some of the nodes in the destination cluster may have become slow, with logs showing prints similar to this:<br/><code>time="2021-02-16T21:51:23Z" level=error msg="Failed to attach cloudsnap internally :774477562361631177 err:Volume with ID: 774477562361631177 not found"</code><br/><br/>**Resolution:** Portworx now fails the restore operation if a volume is deleted before a restore completes. 

## 2.6.4

February 15, 2021

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-18549 | Storage pools with an auto journal partition can now be expanded with a drive resize operation. Use `pxctl service pool expand -o resize-disk` for this operation. |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-18403 | When running vSphere cloud drives, Portworx initialization sometimes failed due to a time out in looking up the disk paths. <br/><br/>**User impact:** Users with VMs containing 2 or more disks that don't show up in the `/dev/disk/by-id` path saw Portworx initialization time out. Portworx looked for the `/dev/disk/by-id` path for each disk for 2 minutes before timing out.<br/><br/>**Resolution:** Portworx will now perform a `udevadm` trigger if it cannot find the disks; the timeout has been removed. |

## 2.6.3

January 15, 2021

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-17546 | Portworx deployments no longer use network ports 6060 and 6061. |
| PWX-16412| Added support for proxy via the `PX_HTTP_PROXY` environment variable for usage-based reporting APIs. |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-17663 | In certain scenarios where the PV to PVC mapping was changed out of band, Portworx showed incorrect "Volume Consumers" under the volume inspect command output. <br/><br/>**User impact:** Users saw incorrect values in the volume inspect command output. <br/><br/>**Resolution:** Portworx now correctly shows values for "Volume Consumers" in the inspect command output. |
| PWX-17155 | Sharedv4 volume consumers were tracked based on nodeID, which changes when a storageless node becomes storage node. <br/><br/>**User impact:** The sharedv4 pods running on a storageless lost access to the mounted sharedv4 volume when that node that was restarted to assume role of storage node. <br/><br/>**Resolution:** Portworx now uses nodeIP instead of nodeID to track the sharedv4 clients as the node IP remains the same when the role changes from storageless to storage node. |
| PWX-17699 | Portworx created the incorrect type of vSphere disks when using Portworx disk provisioning. <br/><br/>**User impact:** Portworx incorrectly parsed the `lazyzeroedthick` disk type provided by users in the vSphere cloud drives spec, and instead created the default `eagerzeroedthick` disks. <br/><br/>**Resolution:** Portworx now correctly parses the spec and created thes correct disk type. |
| PWX-17450 | Volume mount operations inside pods on destination clusters sometimes failed after async DR/Migration. <br/><br/>**User impact:** Users sometimes needed to restart their pods after DR/Migration to correct failed volume mounts. <br/><br/>**Resolution:** These volume mount operations no longer fail. |
| PWX-18100 | Expanding pools with the `add-drive` option created new pools instead of expanding an existing pool. <br/><br/>**User impact:** Users saw new pools created when they were expecting pools to expand in size. <br/><br/>**Resolution:** Portworx now correctly expands pools when the `add-drive` option is presented. |

## 2.6.2.1

January 7, 2021

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-17725 | Migration cloud snapshots sometimes failed due to overlapping extents when being transferred to  the cloud. <br/><br/> **User impact:**  Users saw migrations partially pass. <br/><br/>**Resolution:** Cloud snapshots now handle the transfer of overlapping extents more gracefully to achieve successful cloud snapshots and migrations. |

## 2.6.2

December 7, 2020

### New features

* [Announcing a new command](/reference/cli/cloud-drives-asg/#transfer-cloud-drives-to-a-storageless-node) for transferring a Portworx cloud driveset from one storage node to a storageless node. This command is currently supported only for Google Cloud Platform, and is not supported when Portworx is installed using an internal KVDB. 
* Portworx now allows you to [drain/remove volume attachments](/reference/cli/service/#drain-volume-attachments) from a node through the `pxctl service node drain-attachments` command.
* Portworx [now supports](/operations/key-management/ibm-key-protect/) IBM Hyper Protect Crypto Services (IBM HPCS) as a key management store.

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-17154 | Portworx now supports a `NO_PROXY` environment variable with a licensing code that defines which hosts will _not_ use an HTTP proxy for licensing actions. In addition, `PX_HTTP_PROXY` and `PX_HTTPS_PROXY` environment variables will be _ignored_ when using license-servers and floating licenses, unless you specify the `PX_FORCE_HTTP_PROXY=1` environment variable (i.e. Portworx will assume local-access when working with Floating licenses). | 
| PWX-16410 | Portworx now supports K3s v1.19. | 
| PWX-15234 | You can now delete pending references to credentials from the KVDB.| 
| PWX-16602 | Portworx now allows for a runtime option to disable zero detection for converting zero-filled buffer to discard. |
| PWX-14705 | You can now set the "Sender" for the email alerts as follows: `sv email set --recipient="email1@portworx.com;email2@portworx.com"`. |
| PWX-16678 | Portwox now displays alerts when it overrides the user-provided value for the `maxStorageNodesPerZone` parameter. |
| PWX-16655 | Portworx now supports the `pxctl service pool cache status <pool>` command in all operational modes. The following CLI `pxctl` commands have been moved to only be supported in "pool maintenance" mode and deprecated from "maintenance" mode:<ul><li>`pxctl service pool cache flush <pool-id>`</li><li>`pxctl service pool cache enable <pool-id>`</li><li>`pxctl service pool cache disable <pool-id>`</li><li>`pxctl service pool cache configure <pool-id>`</li></ul> |
| PWX-13377 | Applied the latest OS updates to resolve most of the vulnerabilities detected. |




### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-16137 | A race-condition between Portworx and Docker occurred at startup. <br/><br/>**User impact:** If the node cannot install NFS-packages (i.e. air-gapped host environments), and runs Portworx with the Docker container-runtime, users might encounter a race-condition on node reboot that leads to Docker creating new local volumes instead of using existing Portworx volumes. <br/><br/>**Resolution:** Portworx now detects and prevents Docker from creating new local volumes on node reboot.|
| PWX-16136 | Portworx pods hung when the `dbus` service became unresponsive.<br/><br/>**User impact:** In situations where the `dbus` service running on the host became unresponsive, the Portworx pod wasn't able to upgrade Portworx or propagate configuration changes. <br/><br/>**Resolution:** Portworx now detects when the `dbus` service is unresponsive and fails over to different methods to run the Portworx upgrades or change configuration. |
| PWX-16413 | A harmless warning was sometimes displayed during Portworx installation. <br/><br/>**User impact:** When installed into the staging area, Portworx displayed a warning when setting up the services; these warnings were safe to ignore, but caused unnecessary confusion. <br/><br/>**Resolution:** These warnings are no longer displayed. | 
| PWX-16420 | Portworx pods running on Ubuntu/20.04 nodes did not proxy the `portworx-output.service` logs to the OCI-Monitor display. <br/><br/>**User impact:** On Ubuntu/20.04, the Portworx pods did not proxy the Portworx service logs. This made troubleshooting difficult on Kubernetes deployments that do not provide SSH access to the hosts. <br/><br/>**Resolution:** The Portworx pods now correctly proxy the Portworx service logs. |
| PWX-16418 | When running on Kubernetes nodes using the ContainerD container runtime, Portworx installs and upgrades sometimes failed.<br/><br/>**User impact:** If a Portworx installation or upgrade was interrupted on the nodes using the ContainerD container runtime, further attempts to install or upgrade Portworx sometimes failed until the node was rebooted.<br/><br/>**Resolution:** Portworx now performs a more thorough cleanup before each install and upgrade operation, solving this issue. |
| PWX-16853 | Multipart license expiration was not clear. <br/><br/>**User impact:** When running Portworx on a multipart license (i.e. a license composed out of several parts/ActivationIDs), different parts of the license could expire at different times. The `pxctl status` and `pxctl license list` commands didn't provide any indication that some parts of the license would expire sooner. <br/><br/> **Resolution:** The `pxctl status` and `pxctl license list` commands now display a notice if a part of the license will expire sooner than the overall license. |
| PWX-16769 | License updates sometimes failed to propagate across all nodes. <br/><br/>**User impact:** For users with ETCD as the KVDB and aggressive ETCD compactions, some of the nodes in Portworx cluster skipped the automatic application of updated licenses. Users had to restart the Portworx service to pick up the changes.<br/><br/>**Resolution:** Portworx no longer skips license updates when aggressive ETCD compactions are used.| 
| PWX-16793 | Sometimes Portworx installation timed out while installing NFS packages. <br/><br/>**User impact:** Some host environments may have a large amount of package repositories or have a slow internet connection, which could lead into timeouts while installing NFS services during Portworx installation or upgrade, which ultimately results in inability to use SharedV4 volumes that depend on NFS. <br/><br/>**Resolution:** Portworx installs have been sped up and avoid timeouts with NFS services installations. |
| PWX-17057 | With Linux kernel 4.20.x and above, NFS pool stats are incremented at much slower pace than an hour or it may not increment at all if there are no exports. <br/><br/>**User impact:** With Portworx versions prior to 2.6.2, there may have been false alarms about all NFS threads being busy when there were no exports. <br/><br/>**Resolution:** Portworx no longer processes the NFS pool stats when there are no NFS exports. |
| PWX-16775 | The Google object store did not have pagination for object enumeration, which caused any list call to list everything in the bucket.<br/><br/>**User impact:** Cloudsnap backups and restores failed to start and the request timed out. Listing cloudsnaps through `pxctl` also timed-out.<br/><br/>**Resolution:** Added pagination to object enumeration with the Google object store. |
| PWX-16796 | Proxy username and password were ignored as part of `PX_HTTP_PROXY` on Portworx Essentials causing license renewal to fail. <br/><br/>**User impact:** Portworx essentials clusters went into "license expired" mode with PX_HTTP_PROXY <br/><br/>**Resolution:** Portworx essentials now honors Username and Password fields given as part of `PX_HTTP_PROXY` to successfully make connections with proxy |
| PWX-16072 | When Portworx was installed in PAYG mode, the cluster license expired after being unable to connect to that billing server instead of going into maintenance mode. <br/><br/>**User Impact:** When a PAYG node license expired, users had to recommission the node. <br/><br/>**Resolution:** Portworx PAYG nodes now enter maintenance mode correctly when the billing server is unreachable.| 
| PWX-16429 | Adding a new drive using the `pxctl service drive add` command was failing due to an issue with applying labels on the new pool. <br/><br/>**User impact:** If users wanted to add a new Portworx pool to the node, their command to add a new drive using `pxctl service drive add` failed with an error message about labels being too long. This prevented users from creating new pools. <br/><br/>**Resolution:** The Kubernetes labels are being skipped to be applied to the backend storage pools. |
| PWX-16206 | Portworx failed to correctly detect the value of the `maxStorageNodesPerZone` parameter when running on GKE. <br/><br/>**User impact:** When running on GKE with autoscaling enabled, Portworx did not detect the preferred value to use for the `maxStorageNodesPerZone` parameter for its cloud drives. As a result, Portworx would run without or with an incorrect value for the `maxStorageNodesPerZone` parameter. This resulted in issues when the cluster size was scaled and unintended nodes became become storage nodes. <br/><br/>**Resolution:** The calculation for the value of the `maxStorageNodesPerZone`parameter on the GKE autoscaling min pool size values has been fixed. In addition to this, if the minimum number of nodes in the cluster is lower than the total number of zones, and at least one Portworx node will now be made a storage node in a given zone. |
| PWX-16554 | An incorrect check prevented HA level restoration for volumes with HA level 3 under some conditions during decommission operation.<br/<br/> **User impact:** Decommission operation failed to restore HA level for affected volumes if the volume had HA level 3 under some conditions. </br><br/>**Resolution:** When you decommission a node, Portworx now properly restores the HA level.  | 
| PWX-16407 | DR migration sometimes became stuck in the "Active" state when Portworx restarted while migration was beginning . <br/><br/>**User impact:** Any additional DRs also became stuck and did not finish. <br/><br/>**Resolution:** Portworx now handles this situation better.| 
| PWX-16495 | Currently, the KVDB lock is held over the Inspect API. If the remote node didn't respond, Portworx held the KVDB lock for more than 3 minutes and the node where the Attach was issued asserted. <br/><br/> **User impact:** The Portworx service could assert and restart if it tried to remotely detach a volume from a peer node but the request to do that took more than 3 minutes. <br/><br/>**Resolution:** Portworx will not assert and restart if such a request gets stuck or does not return within a given timeout. |
| PWX-13527 | Internal KVDB startup would fail. This would usually require a wipe of the node and re-install. <br/><br/>**User impact:** Users saw the following error: "Operation cannot be fulfilled on configmaps". <br/><br/>**Resolution**: Portworx will now detect such errors received from Kubernetes and will retry the operation instead of exiting. | 
| PWX-16384 | When a node with a sharedv4 encrypted volume attached was rebooted, the volume was not re-exported over NFS for other nodes to consume. <br/><br/>**User impact:** Since it's an encrypted volume, it couldn't be attached without a passphrase. <br/><br/>**Resolution:** Portworx now triggers a restart of the app, which will reattach the encrypted sharedv4 volume. |
| PWX-16729 | A particular race condition caused an unmount of a sharedv4 volume to succeed without actually removing the underlying NFS mountpoint. <br/><br/>**User impact:** This caused the pod using the sharedv4 volume to be stuck in the "Terminating" state. <br/><br/>**Resolution:** Portworx no longer experiences this race condition. |
| PWX-16715  |In certain cloud deployments, API calls to the instance metadata service or the cloud management portals are blocked or routed through a proxy. <br/><br/>**User impact:** In these cases, Portworx calls to the cloud were blocked indefinitely, causing Portworx to fail to initialize. <br/><br/>**Resolution:** Portworx now invokes all the cloud and instance metadata APIs with a timeout and will avoid getting blocked indefinitely. | 
| PWX-14101  | For certain providers like vSphere, when a cloud drive created by Portworx was deleted out of band, Portworx ignored it, created a new disk, and started as a brand new node. <br/><br/>**User impact:** This caused an issue if there were no additional licenses in the cluster. <br/><br/>**Resolution:** If a disk is deleted out of band or is moved to another datastore in vShpere, Portworx now errors-out and does not create new drives.| 
| PWX-16465 | Portworx held a lock while performing operations in the `dm-crypt` layer, or while making an external KMS API call for encrypted volumes. If either of them took longer than the expected amount of time, Portworx asserted. <br/><br/>**User impact:** In certain scenarios, Portworx restarted while attaching encrypted volumes.<br/><br/>**Resolution:** Portworx will no longer assert if any calls get stuck in the "dm-crypt" layer or if the HTTPS calls to the KMS providers timeout. |
| PWX-15043  |  The `pxctl volume inspect` command output did not show the "Mount Options" field for NFS proxy volumes. <br/><br/>**User Impact:** You could not see the "Mount Options" field for NFS proxy volumes, even if you explicitly provided the mount options while creating such a volume. <br/><br/>**Resolution:** The `pxctl volume inspect` command output now shows the "Mount Options" field for NFS proxy volumes. | 
| PWX-16386 | On certain slower systems, a sharedv4 volume wasn't mounted over NFS as soon as it was exported on the server. <br/><br/>**User impact:** Portworx showed the `access denied by server` error. <br/><br/>**Resolution** Portworx now detects this error scenario and retries the NFS mount. | 
| PWX-17477 | In clusters that have seen more than 3000 unsuccessful node add attempts, Portworx, on addition of another node running the 2.6.x release, encountered a node index overflow. <br/><br/<>**User Impact:** Other nodes in the cluster could dump a core. <br/><br/>**Resolution:** This patch fixes the node index allocation workflow and prevents the new node from joining the cluster. |
| PWX-17206 | A part-probe inside the container took a long time to finish.<br/><br/>**User impact:** Portworx took a long time to reach the "Ready" state after a node restart. <br/><br/>**Resolution:** Portworx now uses a host-based part-probe to resolve this issue. |
| PWX-14925 | Portworx drives showed as `offline` in the `pxctl status` and `pxctl service pool show` commands<br/><br/>**User impact:** When a drive was added to Portworx, the `pxctl status` and `pxctl service pool show` commands showed the drive as offline when the commands were run from the Portworx oci-monitor pod.<br/><br/>**Resolution:** In the oci-monitor pod, `pxctl` now gets the updated information about the newly added drives from the Portworx container. |
| PWX-15984 | A large number of cloudsnap delete requests were stuck pending in the KVDB.<br/><br/>**User impact:** Cloudsnaps were not deleted from the cloud, or cloudsnap delete requests did not make much progress.<br/><br/>**Resolution:** Improvements to cloudsnap delete operations reduce processing times. |
| PWX-16681 | Restore failed when the incremental chain is broken due to either deleted cloudsnaps or deleted local snaps in the source cluster.<br/><br/>**User impact:** Async migration continuously fails.<br/><br/>**Resolution:** To automatically resume async DR, migration now deletes local cloudsnaps if a restore fails, triggering a full backup to fix the issue. |
| OC-196 | An issue with Portworx upgrades from v2.4, 2.5 to v2.6 on Kubernetes with floating licenses caused an excess number of licenses to be consumed. <br/><br/>**User impact:** When upgrading from v2.4, 2.5 to v2.6 on Kubernetes, Portworx temporarily consumed the double amount of license leases.<br/><br/>**Resolution:** The upgrade to Portworx now properly recycles the license leases during the upgrade procedure, no longer consuming more licenses than it should have. |





### Known issues (Errata)

Portworx is aware of the following issues, check future release notes for fixes on these issues:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-17217 | Portworx fails to exit maintenance mode after `drive add` operations showed as "done". <br/><br/>**User impact:** If a user restarts Portworx, the `add drive status` command may return “done” while the md reshape operation was still in progress. Even when the reshape is done, the in-core status of the mountpoint wont change, and users can't exit maintenance mode.<br/><br/>**Recommendations:** If you're stuck in maintenance mode as a result of this issue, you can restart Portworx to clear it. |
| PWX-17531 | A port conflict between containerd and the secure port used by the PVC controller causes the controller to enter the "CrashLoopBackup" mode. <br/><br/>**User Impact:** Users running on Kubernetes clusters using containerd saw their Portworx PVC controller pods enter the "CrashLoopBackup" mode. <br/><br/>**Recommendations:** You can fix this issue by adding the `--secure-port=9031` flag to the `portworx-pvc-controller` deployment, which can be found in the namespace on which you installed Portworx (kube-system by default). If you are using a custom start port for the Portworx installation, add `30` to the configured start port and use that number for the `--secure-port` parameter (e.g. if using 10000, use 10030). |


## 2.6.1.6

November 20, 2020

### Notes

* Portworx licenses for DR are now enabled to work on IBM Cloud.

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-16429 | An issue with applying labels on new pools caused new drive add operations using the `pxctl service drive add` command to fail. <br/><br/>**User impact:** If users tried to add a new Portworx pool to the node, their `pxctl service drive add` command failed with an error message about labels being too long. This prevented users from creating new pools.<br/><br/>**Resolution:** Portworx no longer applies the Kubernetes labels to the backend storage pools. |
| PWX-16941 | Portworx installation failed when users set the `VSPHERE_INSTALL_MODE=local` flag to enable the vSphere cloud drive provisioning feature.<br/><br/>**User impact:** Portworx failed to initialize when used in the mode above.<br/><br/>**Resolution:** Portworx now properly initializes when vSphere cloud drive provisioning is enabled. |

## 2.6.1.5

November 16, 2020

### Notes

* Added support for OCP 4.6.1

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-16796 | For Portworx Essentials users, Portworx ignored the proxy username and password set as part of `PX_HTTP_PROXY`, causing license renewal to fail.<br/><br/>**User impact:** Portworx Essentials clusters entered 'license expired' mode when `PX_HTTP_PROXY` was set. <br/><br/>**Resolution:** Portworx Essentials now honors the `Username` and `Password` fields given as part of `PX_HTTP_PROXY` to successfully make connections with the proxy. |
| PWX-16775 | The Google object store did not have pagination for object enumeration, which caused any list call to list everything in the bucket.<br/><br/>**User impact:** Cloudsnap backups and restores failed to start and the request timed out. Listing cloudsnaps through `pxctl` also timed-out.<br/><br/>**Resolution:** Added pagination to object enumeration with the Google object store. |

## 2.6.1.4

October 30, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-16432 | `Multipathd` configuration files were not correctly set up for blacklisting Portworx devices. <br/><br/> **User impact:** Incorrect entries in the `Multipathd` configuration file caused other devices to be handled incorrectly on the host. <br/><br/> **Resolution:** This fix disables the updates to the `Multipathd` configuration file by default, and adds an option to enable the updates through the `runc` install argument `-enable-mpcfg-update`. |

## 2.6.1.3

October 14, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10434 | When upgrading to the 2.6.x releases on certain platforms where the CPU does not support the SSE4.2 instruction set, Portworx encountered a checksum mismatch on the log file. <br/><br/>**User impact:** The node would go into storageless mode after the upgrade. <br/><br/>**Resolution:** This patch fixes the log replay so that it does the CPU capability check and uses the right checksum type to verify the log checksum. |

## 2.6.1.2

October 9, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-16417 | Portworx would not recognize the multipath devices. <br/><br/>**User impact:** Portworx nodes came up as a storageless node. <br/><br/>**Resolution:** Portworx now properly opens /etc/multipath.conf and recognizes Multipath devices. |

## 2.6.1.1

October 7, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-16013 | On certain kernel versions, such as variants of AKS ubuntu kernel 4.15.0, and under certain conditions, the filesystem IO on the backing storage pool sometimes hung due to a kernel bug. <br/><br/> **User impact:** Impacted Portworx pods displayed a 'healthy' node as 'unhealthy', causing downtime for affected users. <br/><br/> **Impact:** This fix patches the filesystem kernel module in variants of AKS ubuntu kernel 4.15.0 and reinserts the patched kernel module, fixing the issue for users on this kernel. |

## 2.6.1 

October 2, 2020

### New features

* [Introducing Portworx on the AWS Marketplace](/install-portworx/cloud/aws/aws-marketplace/): deploy Portworx from the AWS Marketplace and pay through the AWS Marketplace Metering Service. 

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-16005 | Added support for fetching tokens per vault namespace. | 
| PWX-14307 | Users can instruct Portworx to delete the local snaps created for cloudsnaps after the backup is complete through the `pxctl --delete-local` option. This causes the subsequent backups to be full. | 
| PWX-15987 | The pool expand alert now includes additional information about the cluster ID in the event metrics. |
| PWX-15427 | Volume resize operation status alerts export metrics to Prometheus with additional context, such as: volumeid, clusterid, pvc name, namespace. |
| PWX-13524 | You can now use a network interface for cloudsnap endpoints. This is a cluster-level setting.  |
| PWX-16063 | Introducing a new `px-cache` configuration parameter to control the cache block size for advanced users: `px-runc arg: -cache_blocksize <size>`. | 
| PWX-15897 | Improved Portworx NFS handling in multiple ways: <ul><li>If an unmount operation of the sharedv4 export path fails with `EBUSY`, Portworx restarts the NFS server and tries again.</li><li>If setting the NFS thread count fails - remount the proc FS and set the thread count.</li><li>The NFS thread count now defaults to 128. Portworx now logs an alert if the system is detected as `busy` and requires more NFS threads.|
| PWX-15036 | Traditionally, Portworx installed via Pay-as-you-go or marketplace mode will go into maintenance mode if it is unable to report usage within 72 hours. You can now configure a longer time period, up to 7 days(168 hours), by passing rtOpts `billing_timeout_hours` to the Portworx DaemonSet. If you set an invalid rtOpts value, Portworx returns to 72 hours.| 
| PWX-11884 | Portworx now supports per volume encryption with scaled volumes. DCOS users who use Portworx scaled volumes can provide a volume spec in the following manner to create a scaled volume which uses "mysecret" secret key for encryption: <br/><code>secret_key=mysecret,secure=true,name=myscaledvolume,scale=3</code> |
| PWX-14950 | Portworx now supports the ability to read vSphere usernames and passwords from a Kubernetes secret directly instead of mounting them inside the Portworx spec.| 
| PWX-15698 | Added a new command, `pxctl service node-usage <node-id>`, that displays all volumes/snapshots with their storage usage and exclusive usage bytes for a given node. Since it traverses through the filesystem, this is an expensive command, and should be used with caution and infrequently. <br/><br/>This change also removes support for or deprecates capacity usage of single volume: `pxctl volume usage` is no longer supported. |
| PWX-16246 | px-runc now features the `-cache_blocksize <value>` option, which configures cache blocks size for px-cache. this option supports values of 1MB and above and a power of 2. |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-15130 | OCI-Monitor could have left zombie processes when installing Portworx for the first time.<br/><br/>**User impact:** In most cases these zombies were harmless, but they had the potential to lock up the `yum` package system on CentOS hosts in rare circumstances.<br/><br/>**Resolution:** OCI-Monitor now properly cleans up zombie processes.|
| PWX-16240 | The `ETCD_PASSWORD` environment variable was shown in plaintext on `px-runc` and the OCI-Monitor's logs. <br/><br/>**User impact:** The ETCD_PASSWORD environment variable was shown in plaintext in the Portworx/Kubernetes logs. <br/><br/>**Resolution:** The ETCD_PASSWORD is no longer shown in plaintext in the logs.|
| PWX-15806 | KVDB backups are stored under `/var/lib/osd/kvdb_backup` and one of the internal directories of Portworx where storage is mounted. On storageless nodes, KVDB backup files were not getting rotated from the internal directories since there is no storage.<br/><br/>**User Impact**: The backup files could end up filling the root filesystem of the node.<br/><br/>**Resolution**: Only dump the KVDB backup files under `/var/lib/osd/kvdb_backup` which get rotated periodically. |
| PWX-15705 | Application backups can't work with the newer security model of 2.6.0. <br/><br/>**User impact:** Application backups failed after upgrading to Portworx 2.6.0. <br/><br/>**Resolution:** The auth model now works with the older style of auth annotations. |
| PWX-16006 | Under certain circumstances, Portworx doesn't apply all Kubernetes node labels to storage pools.<br/><br/>**User impact:** PVCs using replica affinity on those labels are stuck in the pending state<br/><br/>**Resolution:**  Portworx now performs the Kubernetes node update later in the initialization process. |
| PWX-15961 | If you reconfigured a network device and attempted to restore a cloud backup on a volume from a snapshot, Portworx tried to use the IP of the previous network device in the restore and the cloud backup failed. <br/><br/>**User impact:** Users saw the following error message: "Failed to create backup: Volume attached on unknown () node", and had to manually attach and detach the volume. <br/><br/>**Resolution:** Portworx now updates the network device when they're reconfigured. |
| PWX-15770 | Portworx sometimes couldn't complete volume export/attach/mount operations for NFS pods before timing-out.<br/><br/>**User impact:** The affected pods failed to deploy. </br><br/>**Resolution:** Portworx no longer retries NFS mount operations in a loop on failure. The timeout for the NFS `unmount` command starts at 2 minutes, and if retried in a loop, an API Mount request can scale up to more than 4 minutes.|
| PWX-15622 | sharedv4 volume mounts timed-out. <br/><br/>**User Impact:** In a slower network or on overloaded nodes, sharedv4 (NFS) volume mounts can timeout and attempt multiple retries. The affected pod never becomes operational and repeatedly shows the `signal: killed` error. <br/><br/>**Resolution:** sharedv4 volume mount operations now wait 2 minutes before timing-out. You can also specify an option to configure the timeout to larger values if required:<br/> `pxctl cluster options update --sharedv4-mount-timeout-sec <value>` |


## 2.6.0.2

September 25, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-16160 | Environment variables were not anonymized. <br/><br/> **User Impact:** Sensitive information regarding secrets may potentially have been printed in the logs. <br/><br/> **Resolution:** Portworx now anonymizes all environment variables. |

## 2.6.0.1

September 22, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-16005 | Added support for fetching tokens per vault namespace |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-15705 | Application backups can't work with the newer security model of 2.6.0. <br/><br/>**User impact:** Application backups failed after upgrading to Portworx 2.6.0. <br/><br/>**Resolution:** The auth model now works with the older style of auth annotations.

## 2.6.0

August 25, 2020

### Notes

* If you're upgrading an auth-enabled Portworx cluster to Portworx 2.6.0, you must upgrade Stork to version 2.4.5. 
* Operator versions prior to 1.4 and Autopilot currently do not support auth-enabled clusters running Portworx 2.6.0. Support for this is planned for a future release

### New features

* [Announcing guest (public) role access](/concepts/authorization/overview/#guest-access-role): the guest role allows your users to access and manipulate public volumes.
* Portworx now features K3s support: deploy Portworx on the K3s distribution.
* [Check out proxy volumes (NFS)](/operations/operate-kubernetes/storage-operations/create-pvcs/create-proxy-volume-pvcs): use this feature to proxy an external NFS share onto your volumes.
* Introducing automatic cluster-wide capacity distribution and balancing. You can also run [rebalance operations manually](/reference/cli/service/#rebalance-storage-across-drives). 

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-12620 | Added support for the [K3s](https://k3s.io) Kubernetes distribution. <br> Limitation: you must use CSI integration to generate / use PVCs. |
| PWX-12587 | Renamed two Prometheus metrics for the floating license server:  <ul><li>`px_alerts_license_lease_has_expired_total` is now `px_alerts_licenseleaseexpired_total`</li><li>`px_alerts_license_leases_expiring_total` is now `px_alerts_licenseleaseexpiring_total`</li></ul> |
| PWX-12477 | Added a new `pxctl` command for rebalancing storage pools. To see the syntax, use `pxctl service pool rebalance --help`. |
| PWX-14465 | When a KVDB node is offline, if there are both storageless and storage nodes available to failover and no nodes are marked as metadata-nodes, Portworx prefers storage nodes to start the KVDB. |
| PWX-12466 | The `--all` option for the `pxctl cloudmigrate start` command has been deprecated. |
| PWX-10590 | Portworx now reports Prometheus metrics related to its KVDB activity. These metrics start with the following prefix: `px_kvdb_`. |
| PWX-13628 | Improved dynamic updates to the Portworx service in Kubernetes environments. <br/>Changing Kubernetes configuration directives, such as `dnsPolicy`, previously did not propagate to the Portworx service (Portworx service needed to be restarted to apply the changes). After the fix, changing the Kubernetes network configuration now automatically propagates to the Portworx service with no restarts required. |
| PWX-14780 | Changed the threshold of repeated alerts from 30 seconds to 2 minutes. |
| PWX-13806 | Added support for cloudsnaps on the Azure government cloud. To enable this, set the environment variable `AZURE_ENVIRONMENT` to `AzureUsGovernmentCloud`. |
| PWX-13437 | Cloudsnaps now allow for `STANDARD_IA` as the storage class for cloudsnap objects with AWS `-s3`. You can specify the storage class while creating credentials. |
| PWX-12151 | You can now add labels to a sharedv4 volume so that the remote mount/client uses NFSv4. For more information, refer to the [Updating volumes using pxctl](/reference/cli/updating-volumes/) article. |
| PWX-11543 | On node startup, each node will raise an alert if it is unable to connect to any of the endpoints of an external etcd. |
| PWX-15226 | Older Vault clients on older Go versions could leak stale connections to the Vault server if authentication with vault fails. Portworx will now cleanup such stale connections. |
| PWX-12274 | The `pxctl service pool delete` command now prints a list of volumes on the pool. |
| PWX-13398 | Added the dashboards showing the replication status to the spec generator. |
| PWX-15047 | Portworx now raises a detailed error event if installed on nodes without disks. |
| PWX-14666 | Improved the recovery time when a storage node goes down and there is another storageless node running in the same failure domain. </br>The recovery will happen only if the machine (VM) for the downed storage node is also deleted, since the storageless node performing the recovery needs to attach the downed node's drives. |
| PWX-12646 | Entering the`pxctl status` command without a token provides confusing information. <!-- ??? -->|
| PWX-14206 | You can now specify runtime MountOptions for Portworx volumes. </br>Added two new fields to Portworx volumes: <ul><li> `mount_options`: Allows you to specify mount-time options for regular block volume mounts.</li><li>`sharedv4_mount_options`: Allows you to specify (NFS) mount-time options for sharedv4 volumes.</li></ul> |
| PWX-14102 | You can now export a sharedV4 volume outside of a Portworx cluster using specific IP addresses. |
|PWX-10207| You can now override a volume's group field using `pxctl`: <br/><br/> `pxctl volume update --group <GROUP> <VOL_NAME>`|
| PWX-11884 | Portworx now supports per-volume encryption with scaled volumes. DCOS users who use Portworx scaled volumes can provide a volume spec in the following manner:</br> `secret_key=mysecret,secure=true,name=myscaledvolume,scale=3`</br> This example creates a scaled volume that uses the "mysecret" secret key for encryption. |
| PWX-15276 | You can now configure the frequency as well as the memory limit at which the Portworx process's memory heap and stack will be dumped: <ul><li>`-rt_opts heapdump_limit_mb=<mb>`</li><li>`-rt_opts heapdump_frequency_min=<interval-in-minutes>`</li></ul> |
| PWX-11967 | The `pxctl status` command now displays which device is being used for the internal KVDB. This will be shown only on the nodes where the internal KVDB is running. |
| PWX-13718 | Portworx now returns a user-friendly error when a sharedv4 mount command times-out instead of `signal: killed`. |
| PWX-12367 | Cloudsnap metrics fields have changed with this release. Previously, the fields were `px_backup_stats_status`, `px_backup_stats_size`, and `px_backup_stats_duration_seconds`. These are replaced with `px_backup_stats_backup_status`, `px_backup_stats_backup_size`, and `px_backup_stats_backup_duration_seconds`. |
| PWX-12271 | Portworx now supports VMware Photon OS 3.0 | 

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-12672 | Due to an accounting issue, I/O gets delayed when a node goes down under a specific condition<br/><br/>**User impact:** I/O may be delayed for a while, but will recover on its own.<br/><br/>**Resolution:** Fix accounting to allow I/O to continue if a node goes down under specific conditions. |
| PWX-14333 | When both the `multipathd` device and the backing device contained the KVDB volume label, Portworx sometimes mistakenly picked up the backing device and failed to mount it. <br/><br/>**User impact:** Portworx failed to come up on the nodes where the internal KVDB was running.<br/><br/>**Resolution:** Portworx now chooses the `multipathd` device when both `multipathd` and the backing device contain the KVDB volume label. |
| PWX-14937 | Added a fix to allow pool expand using the add-disk operation on storage pools that have a journal partition. <br/><br/>**User impact:** If users installed Portworx with cloud drive provisioning using the auto-journal (`-j auto`), expanding the pool with the `add-disk` operation type failed with a "not supported" error message. Users also saw the same error when using Autopilot to expand the pools. <br/><br/>**Resolution:** If Portworx was installed using the auto-journal, it can now expand pools using the `add-disk` operation type. |
| PWX-11545 | Portworx installations with storage pool caching failed to come up after upgrading from 2.3.x to 2.4 or 2.5. <br/><br/>**User impact:** Portworx failed to become operational after upgrading from 2.3.0 to 2.5.0. <br/><br/>**Resolution:** This change fixes the upgrade path in 2.6.0, allowing you to upgrade from 2.3.0 to 2.6.0 without experiencing a failure. |
| PWX-13777 | Kubernetes makes the information about its services available to pods via environment variables. However, each update to environment variables set for a Portworx pod would result in the restart of the Portworx `systemd` service.<br/><br/>**User Impact:** Kubernetes clusters that frequently update services could experience storage interruptions.<br/><br/>**Resolution:** OCI-Monitor now manages the changes to its pod environment variables, so the updates made on the unrelated Kubernetes services will no longer cause restarts of the Portworx `systemd` service. |
| PWX-14160 | The Portworx SDK did not return the list of disks and pools in its response. <br/><br/>**User impact:** Users of the [Portworx SDK](/reference/developer-sdk/) or of the `pxctl -j cluster list` command would see empty `Disks` and `Pools` fields in the response. <br/><br/>**Resolution:** The Portworx SDK handler now populates these 2 missing fields. |
| PWX-13655 | When Portworx went down or restarted, it created detailed logs. On some systems, this operation could take a long time and potentially hang.<br/><br/>**User impact:** Log collection could become unresponsive as a result of this log dump. <br/><br/>**Resolution:** Unnecessary logs were removed, eliminating the possibility for log collection to hang. |
|PWX-13086| When a VM running Kubernetes worker node gets deleted, the Portworx drives get deleted.<br/><br/>**User impact:** When a worker node running Kubernetes worker node gets deleted, and the Portworx disks are still attached to the VM, the default vSphere behavior is the disks also get deleted. This causes Portworx to lose its data disk, and hence users will end up losing the Portworx node. <br/><br/>**Resolution:** For vSphere 6.7.3 and above, create Portworx disks (vmdks) such that they don't get deleted on VM deletion by using the `keepAfterDeleteVm` flag. For lower vSphere versions, the issue persists.|
| PWX-11352 | Mounting both `<dir>` and `<dir>/<subdir>` into the Portworx container could lead to excessive accumulation of disk-mounts.<br/><br/>**User impact:** Mounting both `<dir>` and `<dir/subdir>` in Portworx could cause mountpoints to keep accumulating with every service restart, resulting in host slowdown and (eventually) an unresponsive/hung host system. <br/><br/>**Resolution:** Portworx now checks for overlapping mount directives, and rejects such mount configurations.|
| PWX-12528 | When upgrading Portworx on OpenShift, upgrade operations were blocked. Portworx will no longer make the `/etc/pwx/.private.json` configuration file on the host immutable. <br/><br/>**User impact:** Openshift upgrades could get blocked. <br/><br/>**Resolution:** Portworx now does not set the immutable flag on the /etc/pwx/.private.json |
| PWX-12952 | etcd compaction sometimes broke the propagation of license updates. <br/><br/>**User impact:** If the etcd database was configured to automatically compact, license updates may not propagate to all the nodes in the cluster.<br/><br/>**Resolution:** etcd compaction no longer breaks the license updates. |
| PWX-12466 | The `--all` option for the `pxctl cloudmigrate start` command is deprecated. <br/><br/>**User Impact:** Cloud migration with `-a` option is no longer supported as it would try to migrate internally created snapshots and fail.<br/><br/>**Resolution:** Users can invoke cloud migration it at volume/namespace level to avoid getting into this failure. |
| PWX-14648 | Portworx did not honor the storage class parameter `export_options` during volume creation. <br/><br/>**User impact:** Any export options provided to the storage class were not included when the storage class was used. <br/><br/> **Resolution:** Portworx now parses `export_options` when they are set as storage class parameters. |
| PWX-14941| On volume creation, Portworx formats the volume and then detaches the block device. This detach can sometimes return an `EBUSY` since the format I/O is yet to be completed. Previously, Portworx used to wait for 10 seconds before retrying the detach and did this up to 10 times. <br/><br/>**User impact:** Volume creation sometimes takes more than 10 seconds. <br/><br/>**Resolution:** Portworx now immediately tries to detach and then exponentially backs off if it fails. |
| PWX-13528 | Fixed an issue where vSphere 6.5 deletes disks associated with VM on VM Delete.</br><br/>**User impact:** When Portworx is installed using cloud drives on vSphere 6.5, if users delete the VM directly from vSphere without detaching all the disks, the disks will get deleted causing Portworx to lose quorum. </br><br/>**Resolution:** To ensure the disk is not deleted on VM deletion, you must now provide the `DISK_EXTENSION` environment variable with the disk type as `.pxd` to create a vmdk disk as `.pxd` types. This option currently works with fresh Portworx installations on VMware infrastructure. You cannot upgrade from previous versions to 2.6.0 with `DISK_EXTENSION`.<br/>You do not need to use this on vSphere 6.7u3 and above. |
| PWX-12601 | Restarting the Portworx service while using Objectstore could hang.<br/><br/>**User impact:** Restarting the Portworx service while the uploads local Objectstore was active <!-- I don't understand what exactly was active --> could hang the restart operation for over 5 minutes, and result in the Portworx service being marked as "failed".<br/><br/>**Resolution:** The Portworx service restarts now gracefully and in a timely fashion, stopping active Objectstore uploads. |
|PWX-13542| Portworx running on vSphere using cloud drives fails to come up when it cannot find the path of the attached disk.<br/><br/>**User impact:** Portworx will fail to initialize on the node when it fails to find the device path of the attached disk. <br/><br/>**Resolution:** Portworx will now retry for up to 2 minutes to find the path of the attached disk. |
| PWX-15060 | Portworx may generate cores when user tries to restore an incremental backup that was taken under folder within a bucket. Portworx does not support uploading backups to a folder within the bucket. <br/><br/>**User impact:** Portworx may generate cores till the restore is stopped.<br/><br/>**Resolution:** This change fixes the above issue and returns error to the user rather than generating cores. |
| PWX-13178 | In Azure, a disk can be attached anywhere in an availability set. When using availability sets with the auto-disk provisioning feature in Portworx, storageless nodes did not pick up disks left on terminated storage nodes. Ideally, they should have picked up the disks as they can be attached by any VM in the availability set.<br/><br/>**User impact:** When a storage node is terminated in Azure, any online storageless node should pick up the drives left by the storage node and keep the cluster in quorum. Due to incorrect handling of availability sets, storageless nodes did not pick up the drives.<br/><br/>**Resolution:** Portworx now correctly detects that nodes are part of an availability set and attaches drives left by a storage node in the availability set. |
| PSG-172 | When using CSI in a PKS environment, volume provisioning was not working because an incorrect kubelet host path was used by the CSI sidecar containers.<br/><br/>**User Impact:** CSI PVCs became stuck as "pending" in PKS. <br/><br/> **Resolution:** Fixed the Portworx specs to use the correct kubelet host path when using CSI on PKS. |
| PWX-15341 | Portworx installs on AKS started failing on August 21, 2020 due to an Azure backend change which causes VM disk attach APIs to fail when using VMSS. Until Azure fixes this problem, Portworx calls the Azure disk attach API differently as a workaround.<br/><br/>**User impact:** When using auto disk provisioning in an AKS environment with VMSS, Portworx will not be able to attach Azure Managed Disks and, therefore, not come up at all on the cluster.<br/><br/>**Resolution:** Changed how Portworx invokes the Azure VM Disk attach API. This will ensure that Azure will not throw an error and let the disk attach go through. |
| PWX-12530 | When a node in ASG terminates, it takes a specific amount of time to release the attached drives, and a new replacement node may fail to start due to a license limitation. This caused Portworx to fail correctly, but it did not retry to restart.  Users had to manually restart Portworx.<br/><br/>**User impact:** When a user is using all node licenses in the cluster and wants to replace a Portworx node with a different one (this can happen during Kubernetes or VM upgrades), the new Portworx node tries to join the cluster, but as the old node is not fully terminated yet, Portworx used to exit with a license limit error. As a result, the user has to manually bounce the new node when the old node is terminated so it can join the cluster.<br/><br/>**Resolution:** On license limit error, Portworx now waits for the old node to fully terminate, and release the cloud disks. Once the old node is fully terminated, the new node will restart, and use the drives from the old node without failing with license limit error. |
| PWX-12611 | When using the auto-disk provisioning feature with auto-scaling groups, if the minimum size of an ASG group is zero, and the user specifies a value for the `maxStorageNodesPerZone` parameter, then Portworx overwrites the user-provided value with zero .<br/><br/>**User impact:** Portworx was overriding the user-provided value for the `maxStorageNodesPerZone` parameter with zero, based on the ASG group value. This was causing all new nodes to come up as storage nodes, ignoring the user-provided max storage nodes value.<br/><br/>**Resolution:** When the ASG group minimum value is set to zero, and the user provides a different value, Portworx now does not overwrite the value, honoring the user-provided maxStorageNodesPerZone value.|
| PWX-11338 | With the PX-Cache feature enabled, if any single caching drive goes offline, the entire storage node goes down. <br/><br/>**User impact :** Portworx pools become inaccessible due to single drive failures. <br/><br/>**Resolution:** Portworx now identifies the pool which should be made offline to sustain the failure of single drive.|
| PWX-12670 | If auto journal is used with px-cache, occasionally it is possible for node wipe to fail clearing the md device that was built internal. Manual md stop, followed by device wipefs needed to reinitialize portworx successfully. <br/><br/>**User Impact:** `pxctl sv nw` fails to wipe Portworx signature needed during recommission a node.<br/><br/>**Resolution:** There is a timing dependency from within the kernel to release access to used drives even with Portworx stopped. This was causing node wipe to fail. Internally Portworx now handles the failure and ensure it cleans up the drive with a retry and throws appropriate failure if still in use.|
| PWX-11750 | If you specify a caching device using the `-cache` flag, and the specified device does not exist, then the `-cache` flag  will be ignored, and `pxctl` will not display any cache devices.<br/><br/>**User Impact:** Caching may not be enabled even if the user has provided the `-cache` option at startup.<br/><br/>**Resolution:** Portworx now starts if the cache device is not found, and displays an error message. Note that Portworx will display the error message only if the device is a local  `/dev/...` device and cannot be found. <br/>The following error messages will be displayed in the log:<br/>Using storage device as cache: <br/>Device does not exist: <br/>Warning: ignoring non-existing storage device. |
| PWX-15115 | Fixed an issue where pod termination can get stuck if the user deletes a volume. <br/><br/>**User Impact:** If users delete the entire application stack at once, including the volumes, Portworx sometimes deletes the volume before the pod terminates. In such a situation, the pod will not terminate properly,  because the Portworx volume unmount operation will fail for not being able to find the volume.<br/><br/>**Resolution:** Portworx now correctly handles the situation in which the unmount request is sent for a volume that is no longer present. The pod now terminates properly. |
|PWX-9401| A [bug in Kubernetes 1.13.5 and lower](https://github.com/kubernetes/kubernetes/issues/76340) caused the Portworx volume driver to occasionally save annotations from one PVC into the parameters for another. <br/><br/>**User Impact:** Portworx may have created a PVC with a different group ID than the one set in its annotations. <br/><br/>**Resolution:** Portworx now uses the group value from the PVC annotation that's fetched at runtime from the Kubernetes API during volume creation to ensure the group ID doesn't change.|
| PWX-13362 | Pods may get stuck in the terminating state due to a race condition where the volume gets attached and detached. <br/><br/>**User Impact:** Users may see pods stuck in the terminating state.  <br/><br/>**Resolution:**  You can solve this issue by restarting the pods. |
| PWX-12288  |  Handle a race condition in node decommission which caused the node to be in an inconsistent state where it would stay partially decommissioned and never completed the decommission operation. <br/><br/> **User impact:** Decommissioned node shows in the nodes list. <br/><br>**Resolution:** Due to a race condition in node decommission workflow, Portworx used to leave the decommissioned node in inconsistent state. This has been addressed to avoid this so that decommissioning of node runs to success.|
| PWX-15094 | A nil panic in Portworx sometimes occurred when Portworx successfully created a Vault client while Vault was in an error state.<br/><br/>**User impact:** Portworx returned an error: "too many open file handles" for subsequent vault operations.<br/><br/>**Resolution:** Portworx no longer panics when Vault is experiencing an error. |
| PWX-12088 | Portworx used a version of IBM KeyProtect that caused a kernel panic if multiple threads tried to use it.<br/><br/>**User Impact:** Portworx nodes experiencing a kernel panic restarted, and some apps did not come back online after recovery.<br/><br/>**Resolution:** Portworx now uses IBM KeyProtect library v0.3.5, which solves this problem. |
| PWX-13459 | When using Sharedv4 volumes, if the node where the volume was attached was powered down, daemonset pods on surviving nodes became stuck in the **terminating** state. <br/><br/> **User impact:** Users saw stuck **terminating** pods that Kubernetes was unable to recover. <br/><br/> **Resolution:** Pods now terminate properly. |
| PWX-15250 | Sometimes, when running Portworx with the internal KVdb, the Portworx service doesn't restart on the node where the internal KVdb runs, because the KVdb devices are still mounted. <br/><br/> **User impact:** If you force-terminate the Portworx service, it'll not start on the nodes running  the internal KVdb <br/><br/> **Resolution:** Portworx now successfully restarts. |
| PWX-10674 | Node decommission in certain cases would keep the cloud drives around and not delete them. <br/><br/>**User impact:** Cloud drives used to linger around in cloud after owned node is decommissioned.<br/><br/>**Resolution:** Node decommission now removes the cloud drives which were left undeleted in certain cases. |
| PWX-8385 | Portworx, Inc. discovered a security issue in the method used to provide the secret location to Portworx from a PVC. <br/><br/>**User Impact**: Users could point to a secret in a namespace they could not access in the annotations of a PVC. <br/><br/>**Resolution:** In 2.6, Portworx no longer supports using annotations in the PVC to create volumes. Instead, it uses the same secure method used by CSI and has the StorageClass reference the secret and the secret namespace. |

## 2.5.8

September 1, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-15622 | sharedv4 volume mounts timed-out. <br/><br/>**User Impact:** In a slower network or on overloaded nodes, sharedv4 (NFS) volume mounts can timeout and attempt multiple retries. The affected pod never becomes operational and repeatedly shows the `signal: killed` error. <br/><br/>**Resolution:** sharedv4 volume mount operations now wait 2 minutes before timing-out. You can also specify an option to configure the timeout to larger values if required:<br/> `pxctl cluster options update --sharedv4-mount-timeout-sec <value>` |

## 2.5.7

August 26, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-15341 | Portworx installs on AKS started failing on August 21, 2020 due to an Azure backend change which causes VM disk attach APIs to fail when using VMSS. Until Azure fixes this problem, Portworx calls the Azure disk attach API differently as a workaround.<br/><br/>**User impact:** When using auto disk provisioning in an AKS environment with VMSS, Portworx will not be able to attach Azure Managed Disks and, therefore, not come up at all on the cluster.<br/><br/>**Resolution:** Changed how Portworx invokes the Azure VM Disk attach API. This will ensure that Azure will not throw an error and let the disk attach go through. |

## 2.5.6

August 17, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-14845 | Added a new Portworx install argument (VAULT_NAMESPACE), allowing you to set a global vault namespace for Portworx. Provide this argument as a part of the `px-vault` Kubernetes secret. | 
| PWX-13173 | Added support for providing the `vault-namespace` argument in `pxctl` commands: <ul><li>setting cluster-wide secrets: `/opt/pwx/bin/pxctl set-cluster-key --secret_options=vault-namespace=portworx`</li><li>create volume: `/opt/pwx/bin/pxctl volume create --secure --secret_key=pwx/custom-key --secret_options=vault-namespace=portworx px-vol` </li><li>host attach: `/opt/pwx/bin/pxctl host attach --secret_key=pwx/custom-key --secret_options=vault-namespace=portworx 57416092699073855`</li></ul> |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-14777 | If credentials were deleted from Kubernetes secrets while there were multiple pending references to them in a Portworx cluster, Portworx generated an alert for every pending reference. <br/><br/>**User impact:** Users received an excessive number of alerts. <br/><br/> **Resolution:** Portworx now generates only one alert every hour. Portworx, Inc. advises against deleting credentials if there are schedules managed by Stork. |
| PWX-14034 | Portworx did not read all the vault credentials for Kubernetes authentication methods provided in a Kubernetes secret. <br/><br/>**User impact:** Portworx Vault authentication failed.<br/><br/>**Resolution:** With 2.5.6, Portworx now reads all the Kubernetes authentication related input arguments from the Kubernetes secret and Portworx Vault authentication will succeed. | 
| PWX-14937 | Added a fix to allow pool expand using the add-disk operation on storage pools that have a journal partition. <br/><br/>**User impact:** If users installed Portworx with cloud drive provisioning using the auto-journal (`-j auto`), expanding the pool with the `add-disk` operation type failed with a "not supported" error message. Users also saw the same error when using Autopilot to expand the pools. <br/><br/>**Resolution:** If Portworx was installed using the auto-journal, it can now expand pools using the `add-disk` operation type. |
| PWX-15094 | A nil panic in Portworx sometimes occurred when Portworx successfully created a Vault client while Vault was in an error state.<br/><br/>**User impact:** Portworx returned an error: "too many open file handles" for subsequent vault operations.<br/><br/>**Resolution:** Portworx no longer panics when Vault is experiencing an error. |
| PWX-14924 | A driver bug caused Portworx with PX-Security enabled to send unauthorized requests to the volumes API, which were rejected.<br/><br/>**User impact:** Users were unable to create PVCs, which appeared stuck in the `Pending` state.<br/><br/>**Resolution:** Portworx with PX-Security can once again create PVCs. |

## 2.5.5

July 29, 2020

### Notes

* Portworx 2.5.5 supports OpenShift 4.5.3

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-14102 | You can now export a sharedV4 volume outside of a Portworx cluster using specific IP addresses. |
| PWX-12151 | You can now add labels to a sharedv4 volume so that the remote mount/client uses NFSv4. For more information, refer to the [Updating volumes using pxctl](/reference/cli/updating-volumes/) article. |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-14333 | When both the `multipathd` device and the backing device contain the KVDB volume label, Portworx sometimes mistakenly picked up the backing device and failed to mount it. <br/><br/>**User impact:** Portworx failed to come up on the nodes where internal KVDB was running.<br/><br/>**Resolution:** Portworx now chooses the `multipathd` device when both `multipathd` and the backing device contain the KVDB volume label. |
| PWX-14399 | A bug introduced in Portworx version 2.5.0, broke the documented procedure to perform a KVDB recovery from dump files.<br/><br/>**User Impact:** Users had to manually update the config maps and change the labels on the disk to perform the recovery.<br/><br/>**Resolution:** You can now recover your KVDB from dump files with a single command using the [documented procedure](/concepts/internal-kvdb/#recovery). |
| PWX-14164 | Prior to this version, the `pxctl cloudsnap list -a` command sometimes failed to return all cloudsnaps in an objectstore. <br/><br/>**Resolution:** Portworx now returns all cloudsnaps in an objectstore. |
| PWX-14160 | The Portworx SDK did not return disks and pools in the response. <br/><br/>**User impact:** Users of the [Portworx SDK](/reference/developer-sdk/) or `pxctl -j cluster list` would see empty `Disks` and `Pools` fields in the response. <br/><br/>**Resolution:** The Portworx SDK handler now populates these 2 missing fields. |
| PWX-14648 | Portworx did not honor the storage class parameter `export_options` during volume creation. <br/><br/>**User impact:** any export options provided to the storage class were not included when the storage class was used. <br/><br/> **Resolution:** Parse export_options when they are set as storage class parameters. |
| PWX-13149 | On larger workloads, systems that have lots of overwrites on volumes with a node down sometimes causes Portworx run into an assertion failure while updating the in-core metadata and restart.<br/><br/>**User impact:** Portworx containers restart and continue to work.<br/><br/> **Resolution:** With this fix, Portworx will no longer assert or restart during this scenario.|
| PWX-14389 | On vSphere 6.7 and above, Portworx created thin provisioned vmdks even if the user provided `zeroedthick` as the type. <br/><br/>**User impact:** Users couldn't create thick provisioned vmdks on Portworx. <br/><br/> **Resolution:** Portworx now correctly creates thick provisioned vmdks when `zeroedthick` is specified. |
## 2.5.4.1

July 27, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-13175 | Portworx now supports sharedv4 volumes on hosts running Flatcar OS. |

## 2.5.4

July 2, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-14106 | Added support for a new environment variable: `PX_HTTP_PROXY`. This allows you to specify an HTTP proxy during Portworx installation, permitting sharedv4 volumes and package installation to work properly on air-gapped environments that lacked their own system-wide HTTP proxy. |
| PWX-14186 | Optimized the timeouts on misconfigured air-gapped systems, so after the first failure to install NFS services, Portworx will refrain from attempting to install NFS services again for a period of 10 minutes. |
| PWX-13011 | Reduced Portworx install/upgrade times on air-gapped environments that drop the outbound network traffic (i.e. where "curl portworx.com" command would hang). |

## 2.5.3

June 24, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-13437 | Cloudsnaps now allow for `STANDARD_IA` as the storage class for cloudsnap objects with AWS `-s3`. The storage class can be specified while creating credentials. |
| PWX-13806 | Added support for cloud snaps on Azure government cloud. To enable this, set the enironmement variable `AZURE_ENVIRONMENT` to `AzureUsGovernmentCloud`. |
|PWX-10207|You can now override a volume's group field using pxctl: <br/><br/> `pxctl volume update --group <GROUP> <VOL_NAME>`|
| PWX-13375 | Portworx now accepts a new install argument `--node_pool_label` to determine which Kubernetes node labels to apply to CloudDrive sets. Portworx will only attach those DriveSets to a node if the node label passed through `--node_pool_label` matches the label on the CloudDrive set. |
|PWX-13510| Added a new runtime option `rt_opts kvdb_failover_timeout_in_mins` to configure kvdb offline node failover timeout. The default value is set to 3 minutes.|

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
|PWX-13542| Portworx running on vSphere using cloud drives fails to come up when it cannot find the path of the attached disk.<br/><br/>**User impact:** Portworx will fail to initialize on the node when it fails to find the device path of the attached disk. <br/><br/>**Resolution:** Portworx will now retry upto 2 minutes to find the path of the attached disk.|
|PWX-9401| A [bug in Kubernetes 1.13.5 and lower](https://github.com/kubernetes/kubernetes/issues/76340) caused the Portworx volume driver to occasionally save annotations from one PVC into the parameters for another. <br/><br/>**User Impact:** Portworx may have created a PVC with a different group ID than the one set in its annotations. <br/><br/>**Resolution:** Portworx now uses the group value from the PVC annotation that's fetched at runtime from the Kubernetes API during volume creation to ensure the group ID doesn't change.|
| PWX-13459 | When using Sharedv4 volumes, if the node where the volume was attached was powered down, daemonset pods on surviving nodes became stuck in the **terminating** state. <br/><br/> **User impact:** Users saw stuck **terminating** pods that Kubernetes was unable to recover. <br/><br/> **Resolution:** Pods now terminate properly. |

## 2.5.2.1

June 19, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-13655 | When Portworx went down or restarted, it created detailed logs. On some systems, this operation could take a long time and potentially hang.<br/><br/>**User impact:** Log collection could become unresponsive as a result of this log dump. <br/><br/>**Resolution:** Unnecessary logs were removed, eliminating the possibility for log collection to hang. |
| PWX-13777 | The Portworx pod previously passed unnecessary environment variables to the Portworx service, which sometimes caused it to restart more than it needed to. <br/><br/> **User impact:** When the Kubernetes environment changed, users may have experienced storage interruptions due to these restarts. <br/><br/> **Resolution:** The Portworx pod now passes fewer environment variables to the Portworx service, greatly reducing restarts. |


## 2.3.1.3

June 13, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
|PWX-13086| When a VM running Kubernetes worker node gets deleted, Portworx drives get deleted.<br/><br/>**User impact:** When a worker node running Kubernetes worker node gets deleted and the Portworx disks are still attached to the VM, the default vSphere behavior is the disks also get deleted. This causes Portworx to lose its data disk and hence users will end up losing the Portworx node. <br/><br/>**Resolution:** For vSphere 6.7.3 and above, create Portworx disks (vmdks) such that they don't get deleted on VM deletion by using the `keepAfterDeleteVm` flag. For lower vSphere versions, the issue still persists.|
|PWX-13542| Portworx running on vSphere using cloud drives fails to come up when it cannot find the path of the attached disk.<br/><br/>**User impact:** Portworx will fail to initialize on the node when it fails to find the device path of the attached disk. <br/><br/>**Resolution:** Portworx will now retry upto 2 minutes to find the path of the attached disk.|

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
|PWX-13510| Added a new runtime option `rt_opts kvdb_failover_timeout_in_mins` to configure kvdb offline node failover timeout. Default value is set to 3 minutes.|



## 2.5.0.3

June 12, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-13655 | When Portworx went down or restarted, it created detailed logs. On some systems, this operation could take a long time and potentially hang.<br/><br/>**User impact:** Log collection could become unresponsive as a result of this log dump. <br/><br/>**Resolution:** Unnecessary logs were removed, eliminating the possibility for log collection to hang. |

## 2.5.0.2

June 12, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
|PWX-9401| A [bug in Kubernetes 1.13.5 and lower](https://github.com/kubernetes/kubernetes/issues/76340) caused the Portworx volume driver to occasionally save annotations from one PVC into the parameters for another. <br/><br/>**User Impact:** Portworx may have created a PVC with a different group ID than the one set in its annotations. <br/><br/>**Resolution:** Portworx now uses the group value from the PVC annotation that's fetched at runtime from the Kubernetes API during volume creation to ensure the group ID doesn't change.|

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
|PWX-10207|You can now override a volume's group field using pxctl: <br/><br/> `pxctl volume update --group <GROUP> <VOL_NAME>`|

## 2.5.1.3

June 05, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
|PWX-13086| For vSphere 6.7.3 and above, create Portworx disks (vmdks) such that they don't get deleted on VM deletion. |
|PWX-13542| Fixed in issue where Portworx would fail to come up vSphere using cloud drives when it cannot find the path of the attached disk|
|PWX-13510| Added a new runtime option `rt_opts kvdb_failover_timeout_in_mins` to configure kvdb offline node failover timeout. Default value is set to 3 minutes|

## 2.5.2

May 29, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-12737 | Added support for automatic cloud disk management for non-public Azure clouds like US Government, Germany and China clouds. |
| PWX-13082 | You can now configure the frequency with which Portworx takes its KVDB backups using the `kvdb_dump_frequency_minutes` runtime option. |
| PWX-10216 | Added support for Vault Namespaces. |
| PWX-12603 | When sharedv4 volumes are in use, Portworx uses 16 NFS threads to process them by default. You can change the total number of threads Portworx uses by running the `pxctl cluster options update --sharedv4-threads <num>` command. |
| PWX-12512 | PX-Essential can now refresh licenses through an HTTP or HTTPS proxy. |
| PWX-13116 | Users can now request max backups to be enumerated anywhere between > 0 and < 5000. |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-13189 | A recursive `chmod` operation in Kubernetes 1.18 and lower caused mounts with large block volumes to hang when users set a security context for a pod using `fsGroup`. <br/><br/>**User Impact:** Users with hung mounts would see mount timeouts when creating a pod referencing large block volumes, and pod creation would fail. <br/><br/>**Resolution:** Portworx, Inc. added the `allow_others` storage class label that, when set to true, will apply a permission change to the mount path. This label should only be used for Kubernetes versions lower than 1.18. Users on newer Kubernetes versions can return to using `fsGroup` over the Portworx `allow_others=true` label. |
| PWX-12655 | When Portworx images were uploaded to nodes via docker load command, Docker may not have set the image digest properly.<br/><br/>**User Impact:** When the image digest was not available, OCI-Monitor would not detect manually uploaded Portworx images, and would attempt to pull the Portworx image, potentially failing in air-gapped environments.<br/><br/>**Resolution:** Portworx now has improved image detection, even in cases where image digests are not available. |
| PWX-13171 | The `px_volume_readthroughput`, `px_volume_writethrougput` and `px_volume_iops` metrics did not update.<br/><br/>**User Impact:** Users may have seen values for these metrics reported as zero.<br/><br/>**Resolution:** Portworx once again updates these metrics. |
| PWX-12088 | Portworx used a version of IBM KeyProtect that caused a kernel panic if multiple threads tried to use it.<br/><br/>**User Impact:** Portworx nodes experiencing a kernel panic restarted, and some apps did not come back online after recovery.<br/><br/>**Resolution:** Portworx now uses IBM KeyProtect library v0.3.5, which solves this problem. |
| PWX-12466 | The `--all` option for the `pxctl cloudmigrate start` CLI command has been deprecated. |

## 2.5.1.2

May 28, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-13175 | Portworx now supports sharedv4 volumes on hosts running Flatcar OS. |

## 2.5.1

April 24, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-11638 | Starting with 2.5.1, credentials can be configured without providing secret key or access key to use instance’s IAM capabilities to access cloud provider’s object store. Current support is limited to AWS’s EC2 instance in 2.5.1. |
| PWX-12314 | For {{< pxEssentials >}}, an improvment to the `pxctl status` command now provides the reason for why a license is expired. |


### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-11602 | When Portworx detected an issue with container volumes, for example, if a drive was removed, OCI-Monitor resulted in Portworx pods being stuck in a `CrashLoopBackupOff` state. <br/><br/> **User Impact:** Portworx pods in users' clusters would not recover. <br/><br/> **Resolution:** When Portworx (OCI-Monitor) detects an issue with container mounts, it sends a request to Kuberenetes to reset/reinitialize the Portworx pod, which fixes the issue. |
| PWX-12289 | For the CRI-O container runtime, when OCI-Monitor is set to `ImagePullPolicy:IfNotPresent`, it should pull the {{< pxEnterprise >}} image only when the image is not present on the system. The OCI-Monitor incorrectly identified the image as present while it wasn't. <br/><br/> **User Impact:** Portworx failed to pull the required image and OCI-Monitor failed. <br/><br/> **Resolution:** The OCI-Monitor `ImagePullPolicy` handling now properly pulls images. |
| PWX-12292 | When using OCI-Monitor, Portworx failed to drain its pods when required. <br/><br/> **User Impact:** OCI-Monitor failed to start and upgrade operations failed. <br/><br/> **Resolution:** OCI-monitor now properly starts and Portworx upgrades. |
| PWX-12252 | For CRI-O integrations, the OCI-Monitor did not copy the install logs into its own output. As a consequence, the OCI-Monitor did not parse/retrieve the `INFO: Module version check: Success` install log line, and always triggered the cordoning/draining of the nodes. <br/><br/> **User Impact:** Upgrades to version 2.5.0 stalled on OpenShift and/or CRI-O container-runtime Kubernetes clusters. <br/><br/> **Resolution:** Portworx application cordoning and draining during the upgrade process now works properly, allowing upgrades. |
| PWX-12180 | Portworx didn't send license server alerts for errors packaged into the response body of a valid REST call. <br/><br/> **User Impact:** Users did not see license server alerts for these kinds of errors. <br/><br/> **Resolution:** Portworx now treats these kinds of errors in the same manner as REST errors, and raises alerts accordingly. |
| PWX-11595 | When a Portworx node's storage is down or full, it reports `Not Ready` to Kubernetes to notify the users. In this case, the Portworx node is still available and serves storage in `read-only` mode if it's full, or proxies the storage from other nodes if local storage is not available. |

## 2.5.0.1

April 21, 2020

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-12322 | Portworx failed to start if NFS was in an errored state. <br/><br/> **User Impact:** Users could not start Portworx if NFS was errored. <br/><br/> **Resolution:** Users can now start Portworx if NFS is errored, and Portworx will now raise an alert instead. |

## 2.5

April 3, 2020

### New features

 * Introducing [PX-Central on-premises](https://central.docs.portworx.com/): Deploy your own PX-Central dashboard with Portworx on your own cluster.
 * Introducing [{{< pxEssentials >}}](/concepts/portworx-essentials/): A free Portworx license for prototyping and small production clusters.

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-11777 | Added support to the `pxctl volume list` command, allowing you to list snapshots by node IDs: `pxctl volume list -s --node <node-id>`. |
| PWX-11515 | CLI command flags now use `--node` instead of `--node_id` or `--node-id`.<br/><br/>The following commands were modified:<ul><li>pxctl clouddrive inspect --node</li><li>pxctl clouddrive update-labels --node</li><li>pxctl clouddrive delete --node</li><li>pxctl volume list --node</li></ul> |
| PWX-11464 | Portworx now allows non-root user access to sharedv4 volumes by default. To restrict sharedv4 volume access to the root user, set the `allow_others` label to `false`:<br/><br/>`allow_others=false` |
| PWX-11418 | All cluster and node level alerts will now be raised as Kubernetes events. <br/><br/>NODE alerts: <ul><li>Daemonset model: Events are  raised on the Portworx pod running on that node.</li><li>Operator model: Events are raised on the `StorageNode` object created by the Operator for that node.</li></ul>CLUSTER alerts: <ul><li>Daemonset model: Events are raised on an arbitrary object called `Portworx`</li><li>Operator model: Events are raised on the `StorageCluster` object defined in Kubernetes.</li></ul> |
| PWX-10783 | When Portworx restarts, storageless Portworx nodes will automatically detect any new available storage devices and transition themselves into a storage node. |
| PWX-10756 | When running an internal KVDB without a dedicated drive, `pxctl status` now reports a warning saying that such a configuration is not recommended for a production cluster. |
| PWX-10724 | In cloud, you can now add drives to storageless nodes using the `pxctl` CLI. |
| PWX-10371 | `pxctl status` now reports last known failures if Portworx fails to startup on the node. |
| PWX-9834 | An internal KVDB can now run on storageless nodes. In order to run on storageless nodes, you must provide a `-kvdb_dev` in on-prem clusters, while on cloud Portworx will provision a drive to be used by kvdb. |
| PWX-11774 | A new `pxctl clouddrive listdrives` command allows you to list all the drives in cloud drivesets. On VSphere, this command also lists the datastore for a VMDK in the labels column.


### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10400 | In some situations, a busy volume remained attached even after a pod is terminated in Kubernetes.<br/><br/>**User impact:** Upgrades or other operations relying on the `kubectl drain` command got stuck on a node with these attached volumes.<br/><br/>**Resolution:** Portworx now detaches these busy volumes from terminated Kubernetes pods. |
| PWX-11753 | Portworx sent the nodeID with cluster-level license alerts: `LicenseExpiring` and `LicenseExpired`. <br/><br/>**User impact:** Customers saw the nodeID associated with license alerts when the clusterID would have been more helpful. <br/><br/>**Resolution:** Portworx now reports the ClusterID instead of NodeID with the `LicenseExpiring` and `LicenseExpired` license alerts. |
| PWX-11722 | Portworx raised alerts at the `CLUSTER` level that are more appropriately raised at the `NODE` level. <br/><br/> **User impact:** Users may have seen these alerts at a level they did not expect. <br/><br/> **Resolution:** Portworx now raises the following alerts as `NODE` instead of `CLUSTER` level:<ul><li>NodeStartFailure:</li><li>NodeStartSuccess:</li><li> NodeStateChange:</li><li>NodeJournalHighUsage:</li><li>NodeTimestampFailure:</li><li>NodeJournalFailure:</li><li>NodeDecommissionSuccess:</li><li>NodeDecommissionFailure:</li><li>NodeDecommissionPending:</li><li> NodeInitFailure:</li><li>NodeMarkedDown:</li></ul> |
| PWX-11637 | Cloudsnaps did not work with some AWS S3 endpoints when bucket name being uploaded to had uppercase letters in the name. <br/><br/>**Customer impact:** Snapshot restore operations involving buckets with uppercase letters failed. <br/><br/>**Resolution:** Portworx now supports uppercase letters in bucket names when used with S3 endpoints. |
| PWX-11365 | Portworx only checked the health of the NFS service on the default port: 2049. However, as this port is configurable, these checks failed if the NFS port changed. <br/><br/>**User impact:** Users who configured their NFS to use a port other than the default encountered errors.<br/><br/>**Resolution:** Portworx now checks the health of the NFS service regardless of the port it's running on, and will raise an alert if determines the NFS server is unhealthy. |
| PWX-11280 | Portworx did not update the cloud drive in-progress state after performing a pool expand operation using `resize-disk`. <br/><br/>**User impact:** Users could have seen a misleading output indicating pool expansion was still in-progress from the `pxctl cloud list` command output, even though the operation completed.<br/><br/>**Resolution:** Portworx now correctly reports the cloud drive status after a `resize-disk` operation. |
| PWX-10749 | If all nodes in a cluster were storageless, Portworx failed to properly install. <br/><br/>**User impact:** Users attempting to install Portworx on a cluster with only storageless nodes would be left with an out-of-quorum cluster, and would have to wipe the whole installation and redo it with storage nodes.<br/><br/>**Resolution:** Portworx will no longer form a cluster if it cannot find any storage nodes, and will keep reporting an error until a storage node is added to the cluster. |
| PWX-10711 | On slower systems, Portworx occasionally received an `access denied` error from the NFS, and failed to mount sharedv4 volumes. <br/><br/>**User impact:** Users experiencing this issue had to manually retry the sharedv4 mount operation.<br/><br/>**Resolution:** Portworx now retries mounting a sharedv4 volume if it gets an access denied error. |
| PWX-11643 | Intermittent vCenter API failures occasionally caused Portworx to fail to find its already attached cloud drives.<br/><br/>**User Impact:** Disk operations reliant on the the vCenter API would fail.<br/><br/>**Resolution:** Portworx now automatically retries operations involving the vCenter API before reporting an error. |


## 2.4

March 3, 2020

### New Features

* Introducing the Portworx license server: Add and manage licenses for multiple Portworx clusters from an on-premises license server. UI integration coming soon.

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| PWX-10852 | Improved prometheus metrics for Portworx |


### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10939| A difference between how Portworx calculates license expiration dates using time-zones and how Flexera calculates expiration dates without time-zones caused Portworx to occasionally consider new licenses "expired" on their first day.<br/><br/>**User Impact:** Users with multi-part licenses may have been unable to use their multi-part licenses on the first day they activated them.<br/><br/>**Resolution:** Licenses once again work on the first day they're applied and license expiration dates for multi-part licenses display accurately.|


## 2.3.6

February 29, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

| **Improvement Number** | **Improvement Description** |
|----|----|
| 11377 | Adding support for the 4.18.0-147.5.1.el8_1.x86_64 kernel. |

## 2.3.5

February 19, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
|PWX-11000| Portworx now features a Disaster Recovery plan in the IBM Cloud Marketplace. |
|PWX-11122| Portworx now supports dynamic port range change |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
|PWX-11101| If nodes were decommissioned with pending snapshots, those snapshots contained references to the decommissioned node.<br/><br/>**User impact:** New nodes sometimes failed to come up.<br/><br/>**Resolution:** When a node is now decommissioned, Portworx removes any pending snapshots which had references on the decommissioned node. |
|PWX-10441| Due to a race condition in the logic which handles volume attachments during a Portworx restart, sharedv4 volumes could be  tagged as attached when they were not. <br/><br/>**User impact:** This caused stale entries in `/etc/exports`, which led NFS to error out.<br/><br/>**Resolution:** Portworx no longer experiences a race condition at restart, and no longer creates stale entries in `/etc/exports`. |


## 2.3.4

February 3, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
| PWX-10726 | Portworx can now be installed on OpenShift 4.3 when coupled with [Portworx Operator 1.1.1](https://github.com/libopenstorage/operator/releases). |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10974 | In version 2.3.3, Portworx erroneously showed multi-part licenses as "expired" when the license with the earliest expiration date expired. Despite this incorrect reporting, multi-part licenses did not expire and the cluster continued functioning normally. <br/><br/>**User Impact:** Users may have seen multi-part licenses erroneously marked "expired". <br/><br/>**Resolution:** Portworx now correctly displays multi-part license expiration dates. |
| PWX-10967 | PX-Migrate erroneously indicated success when migrating volumes between clusters with the same internal IP addresses. <br/><br/>**User Impact:** Migrations under these circumstances failed, but users saw Portworx indicate success. <br/><br/>**Resolution:** Px-Migrate now successfully migrates volumes between clusters with the same internal IP addresses. |


## 2.3.3

January 23, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
| PWX-10819 | `replica anti-affinity` rules have been deprecated. <br/><br/>**User impact:** Volume creation may fail if using replica anti-affinity volume placement strategy or when restoring volume using cloud backup configured with such a policy. <br/><br/>**Recommendation:** Remove anti-affinity rules and use affinity rules with NotIn, NotEqual operators to achieve the same effect. |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10400 | In some situations, a busy volume remained attached even after a pod is terminated in Kubernetes.<br/><br/>**User impact:** Upgrades or other operations relying on the `kubectl drain` command got stuck on a node with these attached volumes.<br/><br/>**Resolution:** Portworx now detaches these busy volumes from terminated Kubernetes pods. |
| PWX-10809 | Portworx ignored the `max_drive_set_count` field when deployed in disaggregated mode on cloud deployments.<br/><br/>**User Impact:** If an existing node was terminated and replaced without releasing its storage devices, Portworx sometimes brought a new node online as a storage node, exceeding the `max_drive_set_count` field value.<br/><br/>**Resolution:** Portworx now correctly enforces the `max_drive_set_count` field values. |
| PWX-10627 | Portworx processes license expiration dates based on a combination of {{< pxEnterprise >}} and AddOn licenses. If these licenses expired at different times, Portworx would not accurately report when they would expire. <br/><br/>**User Impact:** Users with these licenses may have had their cluster's node capacity reduced unexpectedly and may not have been able to start their cluster if it exceeded the remaining available license capacity.<br/><br/>**Resolution:** Portworx now aligns the license expiration dates and accurately reports when they expire. |

## 2.3.2

December 18, 2019

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
| PWX-10095 | Portworx now restricts access to SharedV4 volumes to nodes requesting a mount on that volume. |
| PWX-10499 | The `storage pool expansion` operation now supports the `auto` option. |
| PWX-10570 | Portworx now accepts the `VAULT_BACKEND` input argument. When `VAULT_BACKEND` is provided, Portworx uses that version of Vault Backend instead of querying Vault's `sys/mounts/*` directory to fetch all the backends. |
| PWX-10535 | If a SharedV4 Portworx volume must be accessed over NFS outside the Portworx cluster, set the label "allow_all_ips=true" on the volume. This will export the volume on 0.0.0.0/0.0.0.0, which allows you to mount this volume on any node accessible in the network. |
| PWX-10380 | SharedV4 volumes are now enabled when SELinux is enabled on Portworx nodes. If you expect SELinux labels to be propagated from an NFS client to a server, set the `ExportOptions` on a volume to `security_label`. You can use the following command to update the volume option: `pxctl volume update --export_options security_label <vol-name>` |
| PWX-10690 | Portworx now uses hourly usage billing. At end of billing cycle, customers are charged by the number of hours Portworx ran rather than the maximum number of nodes used in a given billing cycle. |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10366 | Portworx did not delete node region or zone values when instructed. <br/><br/>**User Impact:** Portworx continued to show deleted node and region zone labels after users deleted them. This issue persisted over restarts. <br/><br/>**Resolution:** Portworx now properly deletes these labels and replaces them with the `default` value. |
| PWX-10381 | Portworx enabled a license feature intended only for cloud deployments in on-prem clusters. <br/><br/>**User Impact:** This feature transferred licenses of offline storageless nodes to available nodes when running on an on-prem cluster. <br/><br/> **Resolution:** Portworx now only enables this feature when deployed in cloud environments. |
| PWX-10468 | On nodes which were also volume servers and had attached SharedV4 volumes, Portworx did not restart application pods when it entered maintenance mode or was decommissioned. <br/><br/>**User Impact:** Users experienced I/O errors caused by missing application pods. <br/><br/>**Resolution:**  Portworx nodes now delete the application pods to recover the application. |
| PWX-10575 | On AKS clusters with VM scale sets, if a Portworx node with cloud drives failed to bootstrap, detach operations also failed.<br/><br/>**User Impact:** Cloud drives remained attached to non-existent Portworx nodes.<br/><br/>**Resolution:** Portworx now properly detaches cloud drives if it fails to bootstrap a node. |
| PWX-10525 | Portworx frequently queried etcd to retrieve the storage spec and check storage pool status and pending drive operations. <br/><br/>**User Impact:** These frequent queries placed an unnecessary load on etcd, resulting in higher than expected resource usage. <br/><br/>**Resolution:** This fix limits the periodic calls and makes them only when necessary: on a version update. |
| PWX-10455 | A failure during a volume create operation can result in a partially formatted volume. A subsequent attach on this volume will retry the formatting operation. In the case of xfs volumes, this formatting operation can fail if the new operation finds an xfs signature on the volume from the previous incomplete operation. <br/><br/>**User Impact:** Partially formatted xfs volumes could not be attached.<br/><br/>**Resolution:** Portworx now uses the force flag when retrying the format operation. |
| PWX-10657 | The etcdv3 client Portworx uses currently contains the following critical bug: https://github.com/etcd-io/etcd/pull/10911. When connected to a secure etcd cluster, if the first endpoint goes offline, the etcd client does not failover and fails to create a new connection. <br/><br/>**User Impact:** Portworx restarts and does not reconnect to the etcd cluster. <br/><br/>**Resolution:** After restarting, Portworx now reshuffles the list of endpoints so that the etcd client reconnects to the cluster. |
| PWX-10701 | In the `pxctl cluster options update` command, Portworx did not use the configured value associated with the `SnapReservePercent` field for overcommit rules if no label selectors were specified. <br/><br/> **User Impact:** Users could not change from the default `SnapReservePercent` value. <br/><br/> **Resolution:** The `SnapReservePercent` value can now be properly configured. |
| PWX-10685 | Portworx accepted invalid inputs to the `pxctl sv drive add --operation status` command. <br/><br/>**User Impact:** Users adding cloud drives were unable to see the status of their add operations.<br/><br/>**Resolution:** Portworx now allows only device paths in `pxctl service drive add --operation status` command. |
| PWX-10632 | With the new BlueStore backend, Ceph no longer uses an ext4 formatted backend. As a result, Ceph doesn't mount the drives and Bluestore opens the devices without the `o_excl` flag.<br/><br/>**User Impact:** When installing with the `-a` option, Portworx saw the device as "not in use" and picked it up as its storage device. <br/><br/>**Resolution:** Portworx now uses additional filters based on the device name and on-disk signature to prevent this. |


## 2.3.1.2

December 12, 2019

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10681 | For Portworx deployments using an internal KVDB with kubelet running as a docker container,  a crash or other interruption which downs both Docker and Portworx can leave an outdated socket file on the node. On restart, Docker attempts to reconnect to Portworx, but Portworx waits on kubelet causing a cyclic dependency.<br/><br/>**User Impact:** Crashes downing both Portworx and Docker without the chance for cleanup rendered both services unable to recover. <br/><br/>**Resolution:** This fix attempts to address this cyclic dependency. Portworx responds to the outdated socket requests as `not available`, allowing Docker to progress through startup. |



## 2.3.1

November 18, 2019

### New Features

* The `pxctl service pool expand` command is now available, allowing you to expand storage pools by adding drives and consuming unused drive capacity. See the [Expand your storage pool size](/operations/operate-kubernetes/storage-operations/create-pvcs/expand-storage-pool/) section of the documentation for more information.

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
| PWX-10148 | If you're using Portworx on Microsoft Azure, Portworx can now expand storage pools by resizing disks. |
| PWX-10332 | Portworx now provides more descriptive error messages for pool expansion failures. |
| PWX-10442 | Added a new flag to the `volume list` command, allowing you to list your volumes per pool UUID: `pxctl volume list --pool-uid <uuid>` |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10414 | Storage pools sometimes fail to come back online after a disk is added as part of a pool expand operation.<br/><br/>**User Impact:** Impacted storage pools may remain down, impacting apps. |

## 2.3

November 12, 2019

### New Features

* Introducing new ways to control [volume provisioning](/operations/operate-kubernetes/storage-operations/create-pvcs/control-volume-provisioning/): customize provisioning ratios, disable thin provisioning, or disable provisioning entirely.

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
|PWX-10275| The `pxctl service pool update <pool_ID> --labels` command functionality has improved. Previously, entering the command would overwrite any existing labels. Users wishing to add a label would have to keep track of and repeat the existing labels they wanted to persist. With the improved functionality, Portworx now behaves as follows: <ul><li>If the label does not exist, Portworx adds it to the current set</li><li>If label already exists, Portworx replaces its value</li><li>If you pass a label with a blank value, Portworx removes the label</li></ul>|

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
|PWX-10155| Portworx storage pool labels do not inherit Kubernetes node labels. <br/><br/> **User Impact:** PVC creation relying on Kubernetes node labels fails.|
|PWX-10239| Entering the `pxctl service drive add -o status` command with a `--spec` flag included causes Portworx to incorrectly add drives. <br/><br/>**User Impact:** Users entering a status command with the conflicting `--spec` flag can erroneously add new drives. <br/><br/> **Resolution:** With 2.3, Portworx no longer accepts these malformed commands as drive add operations. |
|PSP-1978| Portworx occasionally causes a read/write operation to wait indefinitely on workloads with a large number of overlapping writes. <br/><br/> **User Impact:** Impacted volumes enter a read-only state or become unresponsive. |

## 2.2.0.5

December 19, 2019

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10657 | The etcdv3 client Portworx uses currently contains the following critical bug: https://github.com/etcd-io/etcd/pull/10911. When connected to a secure etcd cluster, if the first endpoint goes offline, the etcd client does not failover and fails to create a new connection. <br/><br/>**User Impact:** Portworx restarts and does not reconnect to the etcd cluster. <br/><br/>**Resolution:** After restarting, Portworx now reshuffles the list of endpoints so that the etcd client reconnects to the cluster. |
| PWX-10456 | Portworx Inc. currently packages filesystem dependencies required for Linux kernels into an archive in the Portworx container. Under this current scheme, Portworx does not contain new versions of Linux kernels released after it in the archive. <br/><br/>**User Impact:** Portworx fails to install on clusters using newer versions of RHEL 8 kernels. <br/><br/>**Resolution:** During installation, Portworx now checks mirrors.portworx.com for the latest filesystem dependencies required for running Linux kernels if it cannot find them locally. |

## 2.2.0.4

December 12, 2019

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10681 | For Portworx deployments using an internal KVDB with kubelet running as a docker container,  a crash or other interruption which downs both Docker and Portworx can leave an outdated socket file on the node. On restart, Docker attempts to reconnect to Portworx, but Portworx waits on kubelet causing a cyclic dependency.<br/><br/>**User Impact:** Crashes downing both Portworx and Docker without the chance for cleanup rendered both services unable to recover. <br/><br/>**Resolution:** This fix attempts to address this cyclic dependency. Portworx responds to the outdated socket requests as `not available`, allowing Docker to progress through startup. |

## 2.2.0.3

December 10, 2019

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10661 | Redacted VMware vSphere environment variable values from Portworx logs. |

## 2.2.0.2

November 27, 2019

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10525 | Portworx periodically queries etcd to retrieve the storage spec and check storage pool status and pending drive operations. This fix limits the periodic calls and makes them only when necessary: on a version update. |

## 2.2.0.1

October 25, 2019

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
| PWX-10125 | The Portworx `pxctl service` CLI command now supports [pool deletion](/reference/cli/service#pxctl-service-pool-delete). |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10204 | For Portworx version 2.2.0 on IBM Cloud: If users install Portworx outside of the catalog, Portworx incorrectly starts the metering agent and cannot report usage to the billing server. <br/><br/> **User Impact:** After 72 hours, users' clusters enter maintenance mode |

## 2.2

September 30, 2019

### New Features

* Introducing [Storage pool caching](/concepts/pool-caching/), this feature is available on new clusters only.
* Portworx now features [stateful application backup and cloning](/operations/operate-kubernetes/storage-operations/stateful-applications/), allowing you new ways to manage your stateful applications.
* Visit [PX-Central](https://central.portworx.com), a place where you can learn all about getting started with Portworx.
* New [jq filtering documentation](/reference/cli/filtering-output-with-jq/) demonstrates how you can filter `pxctl` output.
* The [Portworx CSI](/operations/operate-kubernetes/storage-operations/csi/) driver is now generally available for Kubernetes 1.13 and higher.

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
| PWX-9026 | Previously, to enable sharedv4 volumes while installing Portworx, users were asked to provide the  `ENABLE_SHARED_AND_SHARED_v4` environment variable. <br/><br/>With 2.2, this environment variable is no longer required and sharedv4 volumes are enabled by default. |
| PWX-8165 | The `pxctl cluster provision-status` command with the `--show-labels` flag now displays storage pool labels. |
| PWX-9956 | When encrypting volumes with CSI, users can pass the secret information used for volume encryption through storage class templatized parameters. |
| PWX-9888 | When a Portworx volume is resized, the volume usage metrics are now immediately updated in Prometheus. |
| PWX-9831 | Users can use custom labels to designate nodes as KVdb nodes through the `PX_METADATA_NODE_LABEL` environment variable. |
| PWX-9769 | Users can specify the base path for the Vault secret store using the `VAULT_BASE_PATH` environment variable. |
| PWX-9727 | With 2.2, Portworx raises an alert every time a pool is resized. |
| PWX-9481 | Additional `State` column for the `pxctl clouddrive list` and `pxctl clouddrive inspect` commands makes it easier to see the state of a particular drive. |
| PWX-8951 | The `pxctl` command-line utility now allows users to update the credentials and the cloudsnap schedule for a volume. |
| PWX-9976 | With 2.2, users can update a node's CloudDriveSet labels by running the `pxctl clouddrive update-labels --nodeid <node-id>` command. This is useful for when the `px/metadata-node` label must be set to `true` on a node which is part of an operational cluster. |
| PWX-8534 | The `pxctl cloudsnap list` command provides pagination and the users can now specify filters for listing only certain types of backups. By default, migration-related backups are not displayed. |

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-9991 | Using Talisman to upgrade Portworx from version 2.0.x to 2.1 may create a DaemonSet called `portworx-api` with a wrong IP address. As a result, the DaemonSet is not present on the host network. <br/><br/> **User Impact:** This prevented Stork from finding the Portworx service. In turn, users experienced issues while trying to create cluster pairs.<br/><br/> **Resolution:** With 2.2, the upgrade process creates the DaemonSet with the correct IP address. |
| PWX-9777 | If the size of the timestamp file exceeds 10G, Portworx may unexpectedly restart and become stuck in a restart loop. <br/><br/> **User Impact:** Users could not access volumes. |
| PWX-9765 | A recent update to the IBM IAM service requires every client to set the "Content-Length" header in their HTPP requests. <br/><br/> **User Impact:** This caused users to see an `BXNIM0109E:Property missing or empty.` error. <br/><br/> **Resolution:** With 2.2, Portworx sets the "Content-Length" header in every request. |
| PWX-9842 | Openshift on IKS: Pods may get stuck in the `terminating` state during teardown. <br/><br/> **User Impact:** This left stale paths for the volumes, causing the pod teardown to get stuck. |
| PWX-9964 | Under certain circumstances, the volume delete operation may fail. <br/><br/> **User Impact:** Users may see a `Client.Timeout exceeded while awaiting headers` error, and the delete operation may get stuck for approximately 5 minutes. |
| PWX-9889 | Under certain circumstances, Portworx exposes empty volume usage metrics to Prometheus. <br/><br/> **User Impact:** Users are unable to see the correct volume usage metrics in Prometheus. |
| PWX-9883 | When running Portworx with the internal KVdb, the Portworx service may restart on the online nodes during a KVdb failover. <br/><br/> **User Impact:** Users see an `Etcd cluster not reachable` error. |
| PWX-9855 | Rebooting the internal KVdb nodes within a short interval resulted in the configmap entries getting cleaned up. <br/><br/> **User Impact:** The internal KVdb doesn't start, and the cluster goes out of quorum. <br/><br/> **Resolution:** Check the node labels before performing a KVdb node failover. If the node labels don't allow the node to act as a KVdb node, then don't remove the offline KVdb node from the existing cluster. |
| PWX-9826 | If all the nodes are rebooted at the same time, an application node may try to start the internal KVdb, even though it doesn't have the `px/metadata-node=true` label. <br/><br/> **User Impact:** A storage cloud-drive may get attached to a KVdb node or vice versa. <br/><br/> **Resolution:** Make sure that the KVdb drives are only attached to the designated KVdb nodes, and the application nodes don't attach a `DriveSet` belonging to a KVdb node. |
| PWX-9334 | A bug in the Grafana dashboard caused storageless nodes to display as `down` in Grafana. |
| PWX-10031 | If a migration is performed while only the source cluster is licensed for Disaster recovery, Portworx marks the migration as successful even if the resources are not migrated correctly. <br/><br/> **User Impact:** The migrated volume is empty |
| PWX-10000 | The `pxctl volume usage` command errors out on nodes with more than 100 volumes attached. <br/><br/> **User Impact:** Users see the following error message: `Error: Get http://localhost:9001/v1/osd-volumes/usage/<VOL_ID>: net/http: request canceled (Client.Timeout exceeded while awaiting headers)`. |
| PWX-6894 | For sharedv4 volumes, Portworx provides multi-RW access to the volumes by maintaining a single server and multiple clients. If this type of server goes down, the clients receive an `Unmount` request for the volume. This results in a server with a dangling client reference. <br/><br/> **User Impact:** The sharedv4 volumes might end up attached even if there is no consumer. |
| PWX-9974 | Fixed an issue in which the OpenStorage SDK sends the redirect flag for every request to detach a volume. <br/><br/> **User Impact:** Under certain circumstances, the operation of deleting or detaching a volume could timeout after 5 minutes. |
| PWX-9625 | Installing Portworx using a Helm chart on Kubernetes 1.14.3 fails. <br/><br/> **User Impact:** The user sees several helper pods failing because they try to pull an image which doesn't exist. <br/><br/> **Resolution:** Fixed by replacing the `kubectl` repo with `https://hub.docker.com/r/bitnami/kubectl`. |
| PWX-10061 | When running Portworx on Kubernetes, the upgrade process can reset the custom port settings in the `portworx-service` spec to their default values. <br/><br/> **User Impact:** Users see an `HTTP error 404` error in `kubelet` while trying to mount a volume. |


### Known Issues

Portworx is aware of the following issues, check future release notes for fixes on these issues:

|**Issue Number**|**Issue Description**|**Workaround**|
|----|----|----|
| PWX-10049 | CSI: Due to an issue Kubernetes 1.13, if the Kubelet or Portworx goes offline unexpectedly on a node where a volume is attached, the Kubelet will leave orphaned pod directories under `/var/lib/kubelet/pods/*`. The kubelet logs will report these errors every 2 seconds unless this directory is manually cleaned up. | Move or delete the orphaned pod's directory to stop these logs from showing up. |
| PWX-10057 | CSI: In Kubernetes 1.14 with the Portworx CSI driver, unmount may fail intermittently if a volume is attached to a node where Portworx is down. | If unmount fails, retry once Portworx is back online. |
| PWX-10056 | PX-Security: With security enabled, the `pxctl cloudmigrate status` command returns a blank result even there is cloud migration going on. | Use the `pxctl cloudmigrate status --task_id <your_cloud_migration_task_ID>` command to view the migration status. |
| PWX-8421 | Setting collaborator access on a snapshot using `pxctl` may return an error. | `pxctl` properly updates collaborator access, despite returning an error. |

## 2.1.7

December 12, 2019

### Fixes

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
| PWX-10681 | For Portworx deployments using an internal KVDB with kubelet running as a docker container,  a crash or other interruption which downs both Docker and Portworx can leave an outdated socket file on the node. On restart, Docker attempts to reconnect to Portworx, but Portworx waits on kubelet causing a cyclic dependency.<br/><br/>**User Impact:** Crashes downing both Portworx and Docker without the chance for cleanup rendered both services unable to recover. <br/><br/>**Resolution:** This fix attempts to address this cyclic dependency. Portworx responds to the outdated socket requests as `not available`, allowing Docker to progress through startup. |

## 2.1.5

September 13, 2019

### Fixes
The following issues have been fixed in the 2.1.5 release:

|**Issue Number**|**Issue Description**|
|----|----|
|PWX-9911 |When running PX-DR, old cloudsnaps might not have been deleted from the objectstore since deletions were triggered only on full backups. <br/><br/>**User impact:** Over time, the objectstore on the PX-DR destination cluster may run out of space. <br/><br/>**Resolution:** With 2.1.5, the next DR cleans up old cloudsnaps and frees up space in the objectstore. |
|PWX-9892 |Asynchronous DR causes a large number of alerts composed of long strings, which resulted in high memory usage. <br/><br/>**User impact:** ETCD disk usage was unnecessarily high.|
|PWX-9873 |The DC/OS ACS token used to communicate with DC/OS secrets APIs expires every 5 days, and the auth workflow does not refresh this token when it expires. <br/><br/>**User impact:** This caused users to see an `Unauthorized` error, which required them to restart Portworx.|
|PWX-9811 |In PKS, unset regions impact volume provisioning. <br/><br/>**User impact:** Volumes would be improperly provisioned into a single region. <br/><br/>**Resolution:** With 2.1.5, an unset region will be set to "default". |

## 2.1.4

August 26, 2019

### Fixes
The following issues have been fixed in the 2.1.4 release:

|**Issue Number**|**Issue Description**|
|----|----|
|PWX-9781 |Cloudsnap backup operations may fail during catalog collection. <br/><br/>**User impact:** User operations relying on Cloudsnap may fail. <br/><br/>**Resolution:** With 2.1.4, catalog collection has been disabled. |

## 2.1.3

August 8, 2019

### Improvements
Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
|PWX-8793|In order to migrate volumes encrypted with an AWS KMS cluster-wide secret between clusters, both clusters must have the same cluster-wide secret.<br/><br/>With 2.1.3, Portworx introduces new CLI commands. These commands allow you to dump the cluster-wide secret from one cluster in order to upload the same cluster-wide secret to the destination cluster where encrypted volumes will be migrated. For more information, visit the [Dump and upload cluster-wide secrets](/reference/cli/dump-upload-cluster-wide-secret/) article.|

### Fixes
The following issues have been fixed in the 2.1.3 release:

|**Issue Number**|**Issue Description**|
|----|----|
|PWX-8902 |On older versions of Kubernetes configured to use the CRI-O container runtime on CoreOS/RHEL nodes, volume mount operations failed with the following error message: `selinux is enabled on docker. Disable selinux by removing --selinux-enabled from dockerd arguments`<br/><br/>**User impact:** Kubernetes applications running on this particular configuration attempting to use a shared volume never receive their volume and fail to fully start.<br/><br/>**Resolution:** With 2.1.3, Kubernetes application start as expected.|
|PWX-9610 |Portworx used invalid characters for Prometheus metric labels. <br/><br/>**User impact:** Customers using Prometheus experienced errors when attempting to view metrics.<br/><br/>**Resolution:** With 2.1.3, Portworx replaces invalid '/' characters with '_' characters when serving metrics to Prometheus. |
|PWX-9632 |Portworx occasionally detected a public network interface, causing the internal ETCD to attempt to use a public IP address to communicate over blocked ports.<br/><br/>**User impact:** Nodes were unable to form a quorum.<br/><br/>**Resolution:** With 2.1.3, Portworx no longer detects public network interfaces.|

### Known Issues
Portworx is aware of the following issues, check future release notes for fixes on these issues:

|**Issue Number**|**Issue Description**|**Workaround**|
|----|----|----|
|PWX-9607 | The `pxctl volume usage` command may fail, causing the storage layer to become unresponsive and freezing storage I/O on the nodes where the volume is provisioned.| With 2.1.3, this command has been hidden. <br/><br/>If you're still on 2.1.2, avoid entering this command. If storage does become unresponsive as a result of `pxctl volume usage`, reboot the nodes on which your volume has been provisioned.|

## 2.1.2

July 24, 2019

### Key Features

1. [Cloud drive support for Microsoft Azure](/install-portworx/cloud/azure/aks/)
2. [Enhanced Volume placement strategies for advanced volume provisioning rules](/operations/operate-kubernetes/storage-operations/create-pvcs/volume-placement-strategies/)
3. Support for Red Hat Enterprise Linux 8 with CRI-O

### Enhancements

**PWX-8635** - Add support for the CRI-O container runtime.

_User Impact:_ Portworx has added support for the CRI-O container runtime, with the some log file limitations:

  * The progress bar while downloading images is not available.
  * Progress information while installing Portworx binaries is not available.

**PWX-8665** - Support for optimized restores as a cluster option.

_User Impact:_  Users can now enable optimized restores as a cluster level setting using the CLI.

**PWX-9061** - Add ability to remove path-style enforcement for AWS S3.

_User Impact:_ With 2.1.2, Portworx now supports the disabling of path-style enforcement for S3 with the `--disable-path-style` parameter.

### Key Fixes

**PWX-9352** - Upgrading from 2.0.3.7 to 2.1.1 fails.

_User Impact:_  If you have internal KVDB clusters, upgrading from 2.0.3.7 to 2.1.1 is not supported. {{<companyName>}} recommends upgrading from 2.0.3.7 to 2.1.2.

**PWX-8730** - Allow storageless nodes to join when licenses have been exhausted if offline storageless nodes remain.

_User Impact:_  Previously, rolling upgrades of customer environments in cloud auto-scaling groups may have exceeded licensing quota if storageless nodes were created before offline nodes were removed from the cluster. With 2.1.2, offline nodes no longer count against licensing count.

_Recommendations:_ Upgrade to 2.1.2 to support rolling upgrades in cloud auto-scaling environments. If you are unable to upgrade to 2.1.2, you can work with Portworx support to get temporary licenses that increase the node count until an upgrade to 2.1.2 can be planned.

**PWX-9042** - Secrets can be overwritten with certain commands.

_User Impact:_ with 2.1.2, It is no longer possible to unintentionally overwrite secrets using a combination of the `pxctl secrets set-cluster-key` and `pxctl generate secret` commands.

**PWX-8953** -   (Consul) KVDB does not pass the CA certificate file to the Consul client.

_User Impact:_ With 2.1.2, CAFile is now properly sent to the Consul client.

**PWX-8966** - (ASG) Limit the maximum number of drives you can attach to a node.

_User Impact:_  Previously, attempting to attach too many drives results resulted in errors. With 2.1.2, Portworx introduces a maximum limit of 12 drives per node, and will respond to attempts to add more than 12 drives with the following error: `cannot provide more than 12 number of drives per node`.

**PWX-7374** - AWS SC1 and ST1 EBS volumes are unsupported.

_User Impact:_ It was previously not possible to add SC1 or ST1 EBS volumes using the `pxctl service drive add` command. With 2.1.2, Portworx now supports SC1 and ST1 EBS drive types.

**PWX-8792** - Add backoffs to AWS CloudDrive API calls.

_User Impact:_ With 2.1.2, Portworx now slows calls to the AWS Cloud Drive API at increasingly long intervals if it encounters resource limit errors.

**PWX-8701** - The internal KVDB should use DNS names for peer urls.

_User Impact:_ The internal KVDB tracks peer URLs as potentially ephemeral IP addresses. If the entire cluster becomes unavailable in an outage, Portworx may be unable to reconnect. With 2.1.2, internal KVDB now keeps track of nodes using a DNS, and can therefore reconnect in the event of a total cluster outage.

**PWX-8904** - Introduce timeout on storage requests to avoid possible hung situations when Portworx starts.

_User Impact:_ Previously, Portworx may have failed to start, displaying no active I/O operations. With 2.1.2, storage requests now timeout to avoid possible hung situations on node start.

**PWX-8606** - PVC creation fails if no enforcement type is specified in `VolumePlacementStrategy`.

_User Impact:_  Previously, if you did not specify an enforcement type in `VolumePlacementStrategy`, PVC creation failed. With 2.1.2, Portworx will default to `enforcement: required` if you do not specify an enforcement type.

**PWX-8959** - The Snapshot API does not return a typed error if a snapshot already exists.

_User Impact:_ With 2.1.2, this error message is improved. If you try to create a new snapshot where one already exists, a clearly typed error is now returned.

**PWX-9126** - The `nodiscard` option is impacted by volume resize.

_User Impact:_ Previously, resizing a volume sometimes reset a volume’s `nodiscard` option configuration. With 2.1.2, this has been fixed.

**PWX-8690** - Wipe and upgrade scripts fail on Kubernetes 1.14.

_User Impact:_ Due to a feature deprecation with Kubernetes 1.14, wipe and upgrade scripts did not work. With 2.1.2, The `px-wipe` command now correctly removes Portworx pods on Kubernetes 1.14.

**PWX-9175** - Portworx fails to detect the device path on certain types of AWS ec2 instances.

_User Impact:_ With 2.1.2, Portworx no longer fails to detect the device path on certain types of AWS ec2 instances and operating system combinations.

**PWX-7851** - Storage nodes could be removed by the Kubernetes autoscaler.

_User Impact:_ With 2.1.2, Portworx pods are annotated to prevent node removal by the Kubernetes autoscaler.

**PWX-7493** - Portworx selects undesired network interfaces during autodetection.

_User Impact:_ With 2.1.2, Portworx will avoid selecting the undesired network interfaces during configuration.

**PWX-9053** - The pxctl service node wipe command fails to wipe MDRAID devices.

_User Impact:_ With 2.1.2, MDRAID devices are now correctly wiped with the node wipe command.

**PWX-9046** - Portworx doesn’t recognize MDRAID partitions as journal devices.

_User Impact:_ With 2.1.2, Portworx now recognizes MDRAID partitions as journal devices.

**PWX-9054** - Portworx sometimes fails to detect MDRAID partitions.

_User Impact:_ With 2.1.2, Portworx now detects MDRAID partitions when installed with the `-A` option.

**PWX-8938** - Add “sync” and “noac” sharedv4 mount options.

_User Impact:_ With 2.1.2,  “sync” and “noac” sharedv4 mount options are now available.

**PWX-8893** - Add support for `max_storage_nodes_per_zone` on AKS clusters on Microsoft Azure.

_User Impact:_ With 2.1.2, Portworx supports the max_storage_nodes_per_zone parameter, allowing it to limit storage nodes on AKS availability sets.

**PWX-9263** - Volumes can now be created with the XFS filesystem without needing to provide the "force_unsupported_fs_type" option.

_User Impact:_ Previously, provisioning an XFS volume erroneously created an EXT 4 formatted volume. With 2.1.2, provisioning an XFS volume now creates a properly formatted XFS volume.

**PWX-8712** - Verify completion of a backup before marking it as complete.

_User Impact:_ With 2.1.2, Portworx now verifies that a backup operation has completed before recording a backup as completed.

**PWX-8733** - During upgrade, pods using volume subpaths produce errors.

_User Impact:_ With 2.1.2, pods using volume subpaths are correctly detected and bounced.

**PWX-9476** - OCI-Monitor should not restart the Portworx service unless required.

_User Impact:_ Previously, if the OCI-Monitor restarted Portworx any time it detected a change in the configuration file. With 2.1.2, the OCI-Monitor will only restart Portworx if it’s necessary.

### Errata

**PWX-9473** - Portworx fails to attach a cloud drive to an Azure AKS scale set VM.

_User Impact:_ Portworx fails to attach a cloud drive with the following error messages: `Failed to detach disk...`, and `Failed to attach drive...`

_Recommendations:_ Portworx is working with Microsoft to resolve this issue, in the meantime, {{<companyName>}} recommends deleting the impacted VM manually in Azure and allowing it to redeploy.

## 2.1.1

May 4, 2019

### Key Fixes

* PWX-8668 - Kubernetes: Pods get stuck again on "Terminating 0/1". Failing to unmount shared volumes, because they are not found
* PWX-8652 - AWS: Allow specifying custom tags for EBS volumes in spec
* PWX-8643 - Storage pool does not recover out of storage full condition after deleting volumes
* PWX-8529 - Cloudsnap restore very slow for backups that have dependent incremental with large data sets

## 2.1.0.1

May 22, 2019

### Key Fixes

PWX-8870 - Portworx Metro DR didn't mark a node as offline

## 2.1.0

April 19, 2019

### Key Features and Enhancements

* PX-Security
  * [General Concepts](/concepts/authorization)
  * [Kubernetes](/operations/operate-kubernetes/authorization)
  * [CLI Volume access](/reference/cli/volume-access)
  * [CLI Authorization](/reference/cli/authorization)
  * [CLI Role](/reference/cli/role)
* [PX-DR](/operations/operate-kubernetes/disaster-recovery)
  * [Metro DR](/operations/operate-kubernetes/disaster-recovery/px-metro)
  * [Asynchronous DR](/operations/operate-kubernetes/disaster-recovery/async-dr)
* [Automated application level scheduled snaps and cloudsnaps](/operations/operate-kubernetes/storage-operations/create-snapshots/scheduled)
* [Automated app-consistent cluster to cluster migration](/operations/operate-kubernetes/migration/migration-with-stork/#pre-and-post-exec-rules)
* [Optimized incremental cloudsnap restores](/reference/cli/cloud-snaps/)

### Key Fixes

* PWX-7160 - Fix issues with cloudsnaps to IBM Objectstore
* PWX-7481 - Shared volume failed to detach with an error that the volume is mounted. But the volume was not mounted
* PWX-7650 - Portworx install errors w/ "tar: .: file changed as we read it"
* PWX-7794 - License: Aggressive node-decommissions and "Cluster max capacity" error handling
* PWX-7869 - Cloudsnaps: Handle cloud backup deletes in background
* PWX-7891 - pxctl service nw --all failed to delete multi-path devices
* PWX-7951 - In Kubernetes OCI-Mon restarts during updates may leave Portworx down
* PWX-7963 - Cloudsnap restore for xfs volumes has wrong fs type
* PWX-7968 - Cloudsnaps cleanup fails because of missing start timestamp in metadata
* PWX-7989 - Skip cleanup when a pre-existing node fails to join due to licensing
* PWX-8025 - Alert for storage full is deleted even though the condition still exists
* PWX-8033 - DC/OS portworx-mongo now works with nested folders
* PWX-8055 - pxctl import: copy links after all regular files have been copied
* PWX-8060 - Cloudsnap restores will fail when the volume is heavily fragmented
* PWX-8064 - Reducing the HA level of an aggregated volume may cause any active cloud backups on the volume to fail. New cloud backup can be restarted once the HA level has been reduced on the volume
* PWX-8261 - Startup issue with Debian 9 (4.19.0-0.bpo.2-cloud-amd64)
* PWX-8297 - K8S: OCI-Mon must force-pull px-enterprise before reinstalling incomplete Portworx
* PWX-8311 - IKS: OCI Monitor not starting Portworx when both Docker and ContainerD services running
* PWX-8334 - Fixed install progress-bar
* PWX-8335 - Handle mpath device partitions in nodewipe
* PWX-8403 - Error when trying to mount a sharedv4 volume with encryption. The first pod comes up okay, but second and subsequent pods mount results in failure
* PWX-8472 - OpenShift: Portworx mounts leak. Each portworx-service restart will increase the number of mounts
* PWX-8504 - Cloudsnaps: No incremental backups created for cloud backups that have user tags( created by external schedulers)

### Errata

* PWX-8470: ASG: CLI does not update metadata device name, if after restart device name changes

## 2.0.3.8

### Key Fixes

**PWX-9486** - Changes to Portworx runtime configuration.

_User Impact:_ This fix ensures consistent sync times on the backend.

## 2.0.3.7

June 19, 2019

{{<info>}}
Portworx 2.0.3.7 works with Stork 2.1.2. Here are the [release notes](https://github.com/libopenstorage/stork/releases/tag/v2.1.2) for Stork 2.1.2.
{{</info>}}

### Key Features and Enhancements

**Feature: PWX-7549** - Provide the ability to add multiple drives to a Portworx node in a single command.

_Customer Impact_:  None. This is an enhancement as previously Portworx only allowed only one drive to be added at a time.

_Recommendations_: Add only one drive at a time or upgrade to 2.0.3.7 to be able to multiple drives.

### Key Fixes

**Issue: PWX-9203** - Connections get reset in networks with long idle time with bursty traffic

_Customer Impact_: In very rare cases, in specific network setups, the connections between the node were seen to be disrupted after many hours of idle time.

_Recommendations_: 2.0.3.7 implements improved  connection keep-alives to keep node-to-node connections active in case of long idle times

**Issue: PWX-9126** - If a volume has the 'nodiscard' option set after the volume gets resized, the 'nodiscard' is not retained.

_Customer Impact_: For customers who have used this option to increase the performance, the performance may drop after a volume gets resized.

_Recommendations_: None. Recommend upgrading to 2.0.3.7

**Issue: PWX-9118** - Adding an LVM drive partition as journal device failed.

_Customer Impact_: Adding a journal device which is LVM drive partition will fail but the error is not clear in the logs. In general, it is not recommended to use LVM partitions as journal devices because of the performance limitations of such devices.

_Recommendations_: Don't use LVM partitions as a journal device. Upgrade to 2.0.3.7 for better handling.

**Issue: PWX-9055** - Installing Portworx with an LVM volume that has a similar name to another LVM volume will deactivate the other LVM volume (not used by Portworx). For e.g, if Portworx is asked use to /dev/mapper/volone and there is another LVM volume with the name /dev/mapper/volone1, volone1 would get deactivated.

_Customer Impact_: In this case, if the customer was using the raw LVM volume for another application, that volume will be deactivated.

_Recommendations_: {{<companyName>}} recommends inspecting the system beforehand and to use volume names for LVM volumes that do not overlap with the existing volumes. To completely avoid the issue, Portworx recommends upgrading to 2.0.3.7.

**Issue: PWX-8717** - Adding a journal device to a storage-less node right after a storage device was added results in Portworx crashing and restarting.

_Customer Impact_: When a user tries to add a journal device to a storage-less node right after adding a storage device,  Portworx might crash and restart. In general, there is no application error as Portworx restarts within the 10 min device timeout interval, but there may be a very brief slow down of the I/Os.

_Recomendations_: The workaround is to restart Portworx after a storage device was added and then add a journal device. Upgrading to 2.0.3.7 will eliminate the need for a restart.

**Issue: PWX-9054** - Using the `-A` option during install does not recognize mdraid partitions.

_Customer Impact_: If the storage disks provided to Portworx during install are from mdraid partitions, Portworx will not recognize them and will come up in storageless mode.

_Recommendations_: There is no workaround. Upgrading to 2.0.3.7 will enable adding mdraid partitions

**Issue: PWX-8904** - Restarting Portworx node could result in not being able to complete the startup sequence.

_Customer Impact_: This was seen in environments where there was heavy I/O with shared and non-shared volumes. Also, there were lots of replicas on the node to which a lot of traffic was being routed to. This resulted in a scenario where the internal credits were exhausted. 2.0.3.7 increased the credits and resource allocation.

_Recommendations_: Upgrading to 2.0.3.7 resolves this issue.

**Issue: PWX-9017** - De-couple OCI-mon constraints from Portworx systemd service limits

_Customer Impact_: With previous releases, if the user changes the oci-mon system constraints/limits, it will get passed on to Portworx systemd service.

_Recommendations_: This is addressed in 2.0.3.7 where the oci-mon limits are no longer passed on Portworx systemd service

**Issue: PWX-8846**  - Storage-less node ended up with stuck I/Os and needed to be restarted.

_Customer Impact_: A storage-less node stopped processing I/Os and had to be restarted to have the resources released.

_Recommendations_: Storage-less node must be restarted to release the resources so application I/O can begin to run. Resources used by volumes which are detached when the node was down are not released when the node comes back online. This was resolved so these resources are now properly released to the global internal resource pool in Portworx.

**Issue: PWX-9042** - Do not allow overwriting of secrets

_Customer Impact_: Users can potentially overwrite their encrypted volume secrets if they end up using the same name for the new secret.

_Recommendations_: Users must double-check if they already have the same name that they are trying to add new secrets to the same cluster. 2.0.3.7 implemented a check to look for existing names before accepting a new secret. This prevents the secret from being overwritten.

## 2.0.3.6

May 30, 2019

### Key Fixes

* PWX-8740 - Cloudsnaps: Do not create multiple grpc clients to px-storage
* PWX-8299 - Core with 'concurrent map writes' when object store on the remote cluster was unreachable

## 2.0.3.5

May 28, 2019

### Key Fixes

* PWX-5885 - px-runc install is missing option to specify raidlevel
* PWX-7851 - Add pod annotation to prevent scale down of storage nodes in Kubernetes when using autoscaler
* PWX-8701 - Internal Kvdb: Use DNS names for peer URLs instead of IPs for internal kvdb
* PWX-8712 - Cloudsnaps: Verify uploaded objects before marking backup as done
* PWX-8715 - Node index generation fix to avoid same node index generation
* PWX-8733 - Post upgrade to 2.0.3.4: shared volumes were errored because the server endpoint wasn’t there anymore
* PWX-8917 - OCI-Mon: Portworx service not restarted after cpu/mem limits updated

## 2.0.3.4

April 24, 2019

### Key Fixes

* PWX-8472 - OpenShift: Portworx mounts leak. Each portworx-service restart will increase the number of mounts
* PWX-8529 - Fix Cloudsnap restore performance for backups that have dependent incremental with large data sets
* PWX-8652 - AWS: Allow specifying custom tags for EBS volumes in spec

## 2.0.3.3

April 5, 2019

### Key Fixes

* Portworx available on GOOGLE CLOUD PLATFORM MARKETPLACE https://cloud.google.com/marketplace/
* PWX-8451 - Block adding metadata device when running with internal kvdb
* PWX-8345 - Node wipe and upgrade doesn't work if Portworx is installed in a namespace other than kube-system
* PWX-8045 - Cloudmigrate fails if credentials use a custom bucket name
* PWX-7891 - pxctl service node-wipe --all failed to delete multi-path devices
* PWX-8261 - Allow fresh install of Portworx on Linux Kernel version 4.9.0-7-amd64 and 4.9.0-8-amd64

## 2.0.3.2

March 22, 2019

### Key Fixes

* PWX-8062 - Portworx cluster running on Kubernetes does not report volumes metrics
* PWX-8136 - Disable kvproxy audit as it causes the etcd client to trigger unnecessary API requests
* PWX-8098 - Portworx fails to start after reboot on a system with LVM drives and auto-configured journal device

### Errata

* PWX-8161 - If an LVM partition is added as journal device after node initialization, any subsequent system reboot will need the LVM partition to be made visible before starting Portworx. This can be done by running "partprobe"

## 2.0.3.1

March 19, 2019

### Key Fixes

* PWX-8063 - Startup issue with 4.19.0-0.bpo.2-cloud-amd64
* PWX-8060 - Cloud backup restore fails with JSON unmarshalling error
* PWX-7989 - Fix licensing issue which was leading to reducing the number of nodes allowed in the cluster. Differentiate between NEW and PRE-EXISTING node failing to join the cluster, and do not clean up if PRE-EXISTING nodes were the ones causing the failures.
* PWX-7980 - Do not cleanup CloudDrives when the drives are initialized and have labels
* PWX-7968 - Cloudsnaps cleanup fails because of missing start timestamp in metadata
* PWX-7963 - Cloudsnap restore for xfs volumes has wrong fs type
* PWX-7794 - LIC: Aggressive node-decommissions and "Cluster max capacity" error handling

## 2.0.3

March 8, 2019

### Key Features and Enhancements

* SharedV4 support for DC/OS
* SharedV4 volume encryption support
* Support DC/OS Zookeeper for discovery service when using internal kvdb in DC/OS configurations
* Volume Policy Management Support
* Support Azure Key Vault for Secret Store
* Fix Kubernetes CVE for RUNC

### Key Fixes

* PWX-5657 - Fix a corner case where increasing the replication factor of a volume can take much longer when there are multiple levels of volume clones
* PWX-5762 - Add support for Azure Key Vault
* PWX-6868 - Prometheus framework update  to add Portworx support
* PWX-7448 - Show proper error for incorrect pxctl commands
* PWX-7468 - node-wiper script to wipe the namespace created by Kubernetes secrets
* PWX-7481 - Shared volume failed to detach with an error that the Volume is mounted while Volume was not mounted
* PWX-7485 - Display an appropriate message when cluster-wide diags can not be collected
* PWX-7491 - Drive provisioning fixes for issues where extra drives were created than what was specified in the spec.
* PWX-7512 - Speed up Portworx install in DC/OS clusters by installing in each node in parallel.
* PWX-7513 - In DC/OS, Portworx tasks should restart if they go in LOST state
* PWX-7516 - The portworx-prometheus framework version need to be corrected.
* PWX-7571 - CloudSnap : Restore fails sometimes with "failed to get metadata of the backup from cloud"
* PWX-7595 - Handle spurious storage pool Full/offline condition
* PWX-7596 - Portworx creates node labels for every PVs, causing prometheus federation scraping issues
* PWX-7604 - Anonymize the secrets for Key Management Systems
* PWX-7605 - DCOS Portworx-Prometheus pod replace does not work as expected
* PWX-7619 - Make KVDB URLs optional
* PWX-7628 - Alertmanager does not run after installing Portworx 2.0.2
* PWX-7639 - DCOS Portworx framework should install with default options from config.json.
* PWX-7650 - Portworx install errors w/ "tar: .: file changed as we read it"
* PWX-7656 - Shared v4 failover operation fails if the management and data interface of Portworx service is different
* PWX-7661 - [stork] Snapshot status not being updated for all cloudsnaps in groupsnapshot
* PWX-7686 - Enable Portworx to install in AWS instances when auto journaling is enabled.
* PWX-7743 - Prevent Portworx install if only the journal disk is given in the install script and no data disks were given.
* PWX-7766 - When a groupsnapshot request times out, allow for the snapshot scheduler to retry in the next interval or ask the user to retry if it is a manual request
* PWX-7773 - runc CVE-2019-5736 fix #3169

## 2.0.2.3

March 3, 2019

### Key Fixes

* PWX-7919 - Geography updates loading etcd causing high CPU usage

## 2.0.2.2

February 23, 2019

### Key Fixes

* PWX-7664 - When a node running 1.7 with empty journal log gets upgraded to 2.0 and the upgraded node is restarted, the node doesn't fully restart on next boot

## 2.0.2.1

February 8, 2019

### Key Fixes

* PWX-7510 - Remove any secretes info from the diags
* PWX-7214 - licensing engine config update improvements

## 2.0.2

January 26, 2019

### Key Features and Enhancements

* PWX-7207 - Allow docker with SELinux for newer Kubernetes versions
* PWX-7208 - Google Cloud KMS integration

### Key Fixes

* PWX-6770 - Restart docker apps using shared volumes on DCOS
* PWX-7006 - Cloud migration cancel didn't cancel all the volume migrations
* PWX-7007 - Add an alert when Cloud migration task is canceled
* PWX-7179 - Pool io priority for KOPS io1 volume should be correctly displayed
* PWX-7199 - Enable capacity usage command for centos kernel >= 3.10.0-862
* PWX-7226 - DCOS Portworx: Manually updated values in /etc/pwx/config.json does not persist
* PWX-7267 - Hide unknown/non-handled licenses
* PWX-7271 - 'pxctl secrets gcloud list-secrets' shows unnecessary line in the console output
* PWX-7280 - Logs getting flooded with "18 is not 14len(values)" after upgrading the kernel to 4.20.0-1
* PWX-7304 - Handle journal device "read-only" cases
* PWX-7348 - Handle journal device "offline" cases
* PWX-7364 - Portworx boot stuck at ns mount
* PWX-7366 - Portworx service restart issues including "missing mountpoint", or "cannot open file/directory"
* PWX-7407 - OCI Monitor: Initiates cordoning even when px.ko was not loaded
* PWX-7466 - Kubernetes/Upgrade: Talisman does not support CRI/Containerd

## 2.0.1.1

January 19, 2019

### Key Fixes

* PWX-7431 - Strip the labels on a config map to fit 63 characters.
* PWX-7411 - Portworx does not come up after upgrade to 2.0.1 when an auto-detecting network interface

## 2.0.1

December 20, 2018

### Key Fixes

* PWX-7159 - Persist kvdb backups outside of the host filesystem
* PWX-7225 - AMI based ASG install does not pick up user config
* PWX-7097 - `pxctl service kvdb` should display correct cluster status after nodes are decommissioned
* PWX-7124 - Volume migration fails when the volume has an attached snapshot policy
* PWX-7101 - Enable task ID-based sorting for `pxctl cloudmigrate` commands
* PWX-7121 - Creating a paired cluster results in core files in the destination cluster
* PWX-7110 - Delete paired cluster credentials when the cluster pair is deleted
* PWX-7031 - Cluster migration restore status does not reflect the cloudsnap status when cloudsnap has failed
* PWX-7090 - Core files generated when a node is decommissioned with replicas on the node
* PWX-7211 - Fix daemonset affinity in openshift specs
* PWX-6836 - Don't allow deletion of the Portworx configuration data when the Portworx services are still running in the system
* PWX-7134 - SSD/NVME drives are displayed as STORAGE_MEDIUM_MAGNETIC
* PWX-7089 - Intermittent failures in `pxctl cloudsnap list`
* PWX-6852 - If Portworx starts before Docker is started, the `SchedulerName` field in pxctl CLI shows as N/A
* PWX-7129 - Add an option to improve filesystem space utilization in case of SSD/NVMe drives
* PWX-7011 - Cluster pairing for cluster migration fails when one of the nodes in the destination cluster is down
* PWX-7120 - Cloudsnap restore failures cannot be viewed through `pxctl cloudsnap status`

## 2.0.0.1

December 7, 2018

### Key Fixes

* PWX-7131 - Fix an issue with some of the alerts IDs mismatching with the description as part of the upgrade
  from the 1.x version to 2.0.
* PWX-7122 - Volume restores would occasionally fail when restoring from backups that were done with Portworx 1.x versions.

## 2.0.0

December 4, 2018

### Key Features and Enhancements

* PX-Motion - Migration of applications and data between clusters. Application migration is Kubernetes only.
* PX-Central - Single pane of glass for management, monitoring, and metadata services across multiple
   Portworx clusters on Kubernetes
* Lighthouse 2.0 - Supports PX-motion with connection to Kubernetes cluster for application and namespace migration.
* Shared volumes (v4) for Kubernetes
* Support Cloudsnaps for Aggregated volumes
* ‘Extent’ based cloudsnaps - Restartable Cloudsnaps if a large volume cloudsnap gets interrupted
* Support Journal device for Repl=1 volumes
* PX-kvdb (etcd) supported internally with Portworx cluster deployment

### Key Fixes

* PWX-6458: When decreasing HA of a volume, recover snapshot space unused.
* PWX-5686: Implement accounting and display of space utilized by snapshots and clones.
* PWX-6949: Decommissioned node getting listed from one node in the cluster and not from the other
* PWX-6617: PDM: Dump the cloud drive keys when Portworx loses kvdb connectivity.
* PWX-5876: Volume should get detached when out of quorum or pool down.

### Errata

* PWX-7011: Cluster pair creation failing, because of destination Portworx node is marked down
  Workaround: Restart the Portworx node and attempt the cluster pairing again

* PWX-7041: CloudSnap Backup Failed for Pause/Resume by Portworx Restart - All replicas are down

  Workaround: This is a variant of the previous errata.
  For volume with replication factor set to 1, Cloudsnap backup does not resume after the node with replica goes down.

## 1.7.6

**Release Notes:** February 7, 2019

### Key Fixes

* PWX-7304 - Portworx keeps restarting if journal device made read-only
* PWX-7348 - Portworx keeps restarting, VM reboot after journal device made “offline”
* PWX-7453 - cloudsnap cleanup didn't complete properly in cases where errors were encountered when transmitting the diffs
* PWX-7481 - Shared volume mounts fail when clients connections abruptly lost and not cleaned up properly
* PWX-7600 - Volume mount status might be incorrectly displayed when the node where the volume is attached hits a storage full condition and replicas on that node are moved to a new node


## 1.7.5

January 15, 2019

### Key Fixes

* PWX-7364 Namespace stuck volume issue
* PWX-7299 export pool_status as a stat for prometheus
* PWX-7267 LIC: Hide unknown/non-handled licenses
* PWX-7212 Cloudsnap-Restore: Increase restore verbose level for error cases
* PWX-7179 io1 volume added to KOPS cluster gets displayed as STORAGE_MEDIUM_MAGNETIC
* PWX-7033 Objectstore endpoint failover not happening

## 1.7.4

January 7, 2019

### Key Fixes

* PWX-7292 For all storage errors retry 3 times before making pool offline
* PWX-7291 Detect SSD based pools and mount with nossd if kernel version is less than 4.15
* PWX-7214 LIC: Goroutine leak at license watch re-subscription
* PWX-7143 LIC: Should hard-code "absolute maximums" into License evaluations
* PWX-7142 LIC: SuperMicro misinterpreted as VM [roblox]

## 1.7.3

December 13, 2018

### Key Features and Enhancements

* Provide a runtime option to enable more compact data out of flash media to avoid disk fragmentation

### Key Fixes

* Fix an issue with NVMe/SSD disks being shown as Magnetic disks

## 1.7.2

December 5, 2018

### Key Features and Enhancements

* Default queue depth for all volumes (new, coming from older release) set to 128
* Advanced runtime options for write amplification reduction

### Key Fixes

* PWX-6928: Store bucket name in cloudsnap object
* PWX-6904: Fix bucket name for cloudsnap ID while reporting status
* PWX-7071: Do not use GFP_ATOMIC allocation

## 1.7.1.1

November 7, 2018

### Key Fixes

* Fix to add/remove node labels in Kubernetes to indicate where volume replicas are placed

## 1.7.1

November 7, 2018

### Key Features and Enhancements

* Restart docker containers using shared volumes for DC/OS to enable automatic re-attach of the containers on Portworx upgrades
* Preserve Kubernetes agent node ids across agent restarts when Kubernetes agents are running statelessly in
  auto-scaling based environments

## 1.7.0

November 3, 2018

### Key Features and Enhancements

* IBM Kubernetes Service (IKS) Support
* IBM Key Protect Support for Encrypted Volumes
* Containerd runtime Interface (CRI) support
* Automatic VM Datastore provisioning for CentOS ESXi VMs
* Tiered Snapshots for storing volume snapshots on only lower-cost media
* Encryption support for shared volumes

### Key Fixes

* PWX-6616 - Fix shared volume mounts going read-only kubernetes in few corner cases
* PWX-6551 - px_volume_read_bytes and px_volume_written_bytes are not available in 1.6.2
* PWX-6479 - Debian 8: Portworx fails to come up if sharedv4 is enabled
* PWX-6560 - PVC creation fails with "Already exists" perpetually
* PWX-6527 - Clean up orphaned volume paths as PVC are attached and detached over a period of time
* PWX-6425 - Cloudnsap schedule option to do full backup always.
* PWX-6408 - Node alerts: Include hostname/IP in addition to node id
* PWX-5963 - Report volumes with no snapshots

## 1.6.1.4

October 19, 2018

This is a minor patch release with the following fixes/enhancements.

* PWX-6655 - Fix to allow storageless nodes to reuse their node ids in Kubernetes
* PWX-6410 - Fix a bug where Portworx may detach unused loopback devices that are not owned by Portworx on restarts.
* PWX-6713 - Allow update of per volume queue depth

## 1.6.1.3

October 26, 2018

 This is a minor patch release with the following fixes/enhancements.

 * PWX-6697: Add support for automatic provisioning of disks on VMware virtual machines on non-Kubernetes clusters and Kubernetes clusters without vSphere Cloud Provider

## 1.6.1.2

October 23, 2018

This is a minor patch release with the following fixes/enhancements.

* PWX-6567 - Provide a parameter to disable discards during volume create
* PWX-6559 - Provide the ability to map services listening on port 9001 to another port

## 1.6.1.1

October 11, 2018

This is a minor patch release with fixes issues around volume unmounts as well as pending commands to docker.

* PWX-6494 - Fix rare spurious volume unmounts of attached volumes in case of Portworx service restart under heavy load
* PWX-6559 - Add a timeout for all commands to docker so they timeout if docker hangs or crashes.

## 1.6.1

October 2, 2018


### Key Features and Enhancements

* Per volume queue depth to ensure volume level quality of service
* Large discard sizes up to 10MB support faster file deletes. NOTE: You will need a px-fuse driver update to use
  this setting.  Portworx 1.6.1 will continue to work with old discard size of 1MB if no driver update was done. This is a
  backward compatible change
* Enable option to always perform a full clone back up for Cloudsnap
* Reduce scheduled snapshot intervals to support snapping every 15 mins from the current limit of 1 hour

### Key Fixes

* Fix replica provisioning across availability zones for clusters running on DC/OS in a public cloud

## 1.6.0

September 20, 2018

### Key Features and Enhancements

* OpenStorage SDK support. Link to [SDK](https://libopenstorage.github.io/w/)
* Dynamic VM datastore provisioning support Kubernetes on vSphere/ESX environment
* Pivotal Kubernetes Service (PKS) support with automated storage management for [PKS](/install-portworx/on-premises/install-pks/)

### Errata

* PWX-6198 - SDK Cloud backup and credentials services is still undergoing tests
* PWX-6159 - Intermittent detach volume error seen by when calling the SDK Detach call
* PWX-6056 - Expected error not found when using Stats on a non-existent volume


## 1.5.1

September 14, 2018

### Key Fixes

* PWX-6115 - Consul integration fixes to reduce CPU utilization
* PWX-6049 - Improved detection and handling cloud instance store drives in AWS
* PWX-6197 - Fix issues with max drive per zone in GCP
* When a storagless node loses connectivity to the remaining nodes, it should bring itself down.
* PWX-6208 - Fix GCP provider issues for dynamic disk provisioning in GCP/GKE
* PWX-5815 - Enable running `pxctl` from oci-monitor PODs in Kubernetes
* PWX-6295 - Fix LocalNode provisioning pattern when provisioning volumes with greater than 1 replication factor
* PWX-6277 - Portworx fails to run sharedv4 volume support for Fedora
* PWX-6268 - Portworx does not come up in Amazon Linux V2 AMIs
* PWX-6229 - Portworx does not initialize fully in a GKE multi-zone cluster during a fresh install


## 1.5.0

August 21, 2018

{{<info>}}
Important note: Consul integration with 1.5.0 has a bug which results in Portworx querying a Consul Cluster too often for a non-existent key. We will be pushing out a 1.5.1 release with a fix by 08/31/2018
{{</info>}}

### Key Features and Enhancements

* Eliminate private.json for stateless installs
* Handle consul leader failures when running with consul as the preferred k/v store
* When a node is offline for longer than the user-configured timeout, move the replicas in that node out to
  other nodes with free space
* Improvements to AWS Auto-scaling Group handling with KOPS
* Lighthouse Volume Analyzer View Support
* Enable volume resize for volumes that are not attached
* Periodic, light-weight pool rebalance for proactive capacity management

### Key Fixes

 * PWX-5800 - In AWS Autoscaling mode, Portworx nodes with no storage should always try to attach available drives on restart
 * PWX-5827 - Allow adding cloud drives using pxctl service drive add commands
 * PWX-5915 - Add PX-DO-NOT-DELETE prefix to all cloud drive names
 * PWX-6117 - Fix `pxctl cloudsnap status --local` command failing to execute
 * PWX-5919 - Improve node decommission handling for volumes that are not in quorum
 * PWX-5824 - Improve geo variable handling for kubernetes and DC/OS
 * PWX-5902 - Support SuSE CaaS platform
 * PWX-5815 - Enable diags collection via oci-monitor when shell access to the minions not allowed
 * PWX-5816 - Incorrect bucket names will force a full backup instead of incremental backup
 * PWX-5904 - Remove db_remote and random profiles from io_profile help
 * PWX-5821 - Fix panics seen zone and rack labels are supplied on volume create



## 1.4.2.2

August 11, 2018

This is a patch release that adds the capability to switch from shared to sharedv4 one volume at a time. Please contact Portworx support before switching the volume types.


## 1.4.2

July 21, 2018

### Key Features and Enhancements

* Use [PX-Central](https://central.portworx.com) for Kubernetes spec generation.

### Key Fixes

* PWX-5681 - Portworx service to handle journald restarts
* PWX-5814 - Fix automatic diag uploads
* PWX-5818 - Fix diag uploads via `pxctl service diags` when running under Kubernetes environments

## 1.4.0

July 4, 2018

If you are on any of the 1.4 RC builds, you will need to do a fresh install. Please reach out to us at support@portworx.com or on the slack to help assess upgrade options from 1.4 RC builds.

All customers on 1.3.x release will be able to upgrade to 1.4

All customers on 1.2.x release will be able to upgrade to 1.4 but in a few specific cases might need a node reboot after the upgrade. Please reach out to support for help with an upgrade or if there are any questions if you are running 1.2.x in production.

### Key Features and Enhancements

* 3DSnaps - Ability to take [application-consistent](/operations/operate-kubernetes/storage-operations/create-snapshots)
  snapshots cluster wide (Available in 05/14 GA version)
  * Volume Group snapshots - Ability to take crash-consistent snapshots on group of volumes based on a user-defined label
* GCP/GKE automated disk management based on [disk templates](/install-portworx/cloud/gcp/gke/)
* [Kubernetes per volume secret support](/operations/operate-kubernetes/storage-operations/create-pvcs/create-encrypted-pvcs) to enable
  volume encryption keys per Kubernetes PVC and using the Kubernetes secrets for key storage
* DC/OS vault integration - Use Vault integrated with DC/OS
* Support Pool Resize - Available in Maintenance Mode only
* Container Storage Interface (CSI) [Tech Preview](/operations/operate-kubernetes/storage-operations/csi)
* Support port mapping used by Portworx from 9001-9015 to a custom port number range by passing the starting
  port number in [install arguments](/install-portworx/install-with-other/docker/standalone)
* Provide ability to do a [license transfer](/reference/knowledge-base/licensing/licensing-operations) from one cluster to another cluster
* Add support for [cloudsnap deletes](/reference/cli/cloud-snaps#deleting-a-cloud-backup)

### Key Fixes

* PWX-5360 - Handle disk partitions in node wipe command
* PWX-5351 - Reduce the `pxctl volume list` time taken when a large number of volumes are present
* PWX-5365 - Fix cases where cloudsnap progress appears stopped because of time synchronization
* PWX-5271 - Set default journal device size to 2GB
* PWX-5341 - Prune out trailing `/` in storage device name before using it
* PWX-5214 - Use device UUID when checking for valid mounts when using device-mapper devices instead of the device names
* PWX-5242 - Provide facility to add metadata journal devices to an existing cluster
* PWX-5287 - Clean up px_env variables as well when using node wipe command
* PWX-5322 - Unmount shared volume on shared volume source mount only on Portworx restarts
* PWX-5319 - Use excl open for open device checks
* PWX-4897 - Allow more time for the resync to complete before changing the replication status
* PWX-5295 - Fix a nil pointer access during cloudsnap credential delete
* PWX-5006 - Tune data written between successive syncs depending on ingress write speed
* PWX-5203 - Cancel any in-progress ha increase operations that are pending on the node if the node is decommissioned
* PWX-5138 - Add startup options for air-gapped deployments
* PWX-4816 - Check for and add lvm devices when handling -a option for device list
* PWX-4609 - Allow canceling of replication increase operations for attached volumes
* PWX-4765 - Fix resource contention issues when running heavy load on multiple shared volumes on many nodes
* PWX-5039 - Fix Portworx OCI uninstall when shared volumes are in use
* PWX-5153 - In Rancher, automatically manage container volume mounts if one of the cluster node restarts

## 1.3.1.4

May 9, 2018

This is a minor update that improves degraded cluster performance when one or more nodes are down for a long time and brought back online that starts the resync process

## 1.3.1.2

May 2, 2018

This is a minor update to fix install issues with RHEL Atomic and other fixes.

* RHEL Atomic install fixes
* Clean up any existing diag files before running the diags command again
* `pxctl upgrade` fixes to pull the latest image information from install.portworx.com
* improvements in attached device detection logic in some cloud environments

## 1.3.1.1

April 16, 2018

This is a minor update to the previous 1.3.1 release

* Fix to make node resync process yield better to the application I/O when some of the nodes are down for a longer period of time and brought back up thereby triggering the resync process.

## 1.3.1

April 13, 2018

This is a patch release with shared volume performance and stability fixes

#### Key Fixes

* Fix namespace client crashes when client list is generated when few client nodes are down.
* Allow read/write snapshots in Kubernetes annotations
* Make adding and removing Kubernetes node labels asynchronous to help with large number volume creations in parallel
* Fix Portworx crash when a snapshot is taken at the same time as a node being marked down because of network failures
* Fix nodes option in docker inline volume create and supply nodes value as semicolon-separated values

## 1.3.0.1

April 6, 2018

This is a patch update with the following fix

* PWX-5115 - Fix `nodes` option in [docker inline volume create](/operations/operate-other/operate-docker/volume-plugin) and supply nodes value as semicolon separated values

## 1.3.0

March 6, 2018

_**Upgrade Note 1**_: Upgrade to 1.3 requires a node restart in non-Kubernetes environments. In Kubernetes environments, the cluster does a rolling upgrade

_**Upgrade Note 2**_: Ensure all nodes in Portworx cluster are running 1.3 version before increasing replication factor for the volumes

_**Upgrade Note 3**_: Container information parsing code has been disabled and hence the PX-Lighthouse up to 1.1.7 version will not show the container information page. This feature will be back in future releases and with the new lighthouse

### Key Features and Enhancements

* Volume create command additions to include volume clone command and integrate snap commands
* Improved snapshot workflows
  * Clones - full volume copy created from a snapshot
  * Changes to snapshot CLI.
  * Creating scheduled snapshots policies per volume
  * _**Important**_ From 1.3 onwards, all snapshots are read-only. If the user wishes to create a read/write snapshot, a volume clone can be created from the snapshot
* Improved resync performance when a node is down for a long time and restarted with accumulated data in the surviving nodes
* Improved performance for database workloads by separating transaction logs to a separate journal device
* Added Portworx signature to drives so drives cannot be accidentally re-used even if the cluster has been deleted.
* Per volume cache attributes for shared volumes
* https support for API end-points
* Portworx Open-Storage scaling groups support for AWS ASG - Workflow improvements
  * Allow specifying input EBS volumes in the format “type=gp2,size=100”. \(this is documented\)
  * Instead of adding labels to EBS volumes, Portworx now stores all the information related to them in kvdb. All the EBS volumes it creates and attaches are listed in kvdb and this information is then used to find out EBS volumes being used by Portworx nodes
  * Added command `pxctl cloud list` to list all the drives created via ASG
* Integrated kvdb - Early Access - Limited Release for small clusters less than 10 nodes

#### New CLI Additions and changes to existing ones

* Added `pxctl service node-wipe` to wipe Portworx metadata from a decommisioned node in the cluster
* Change `snap_interval` parameter to `periodic` in `pxctl volume` commands
* Add schduler information in `pxctl status` display
* Add info about cloud volumes CLI [Kubernetes](/cloud-references/auto-disk-provisioning/gcp), [others](/install-portworx/cloud/aws/aws-asg)
* `pxctl service add --journal -d <device>` to add journal device support

### Key Fixes

* PWX-4518 - Add a confirmation prompt for `pxctl volume delete` operations
* PWX-4655 - Improve “PX Cluster Not In Quorum” Message in `pxctl status` to give additional information.
* PWX-4504 - Show all the volumes present in the node in the CLI
* PWX-4475 - Parse io\_profile in inline volume spec
* PWX-4479 - Fix io\_priority versions when labeling cloudsnaps
* PWX-4378 - Add read/write latency stats to the volume statistics
* PWX-4923 - Add vol\_ prefix to read/write volume latency statistics
* PWX-4288 - Handle app container restarts attached to a shared volume if the mount path was unmounted via unmount command
* PWX-4372 - Gracefully handle trial license expiry and Portworx cluster reinstall
* PWX-4544 - Portworx OCI install is unable to proceed with aquasec container installed
* PWX-4531 - Add OS Distribution and Kernel version display in `pxctl status`
* PWX-4547 - cloudsnap display catalog with volume name hits “runtime error: index out of range”
* PWX-4585 - handle kvdb server timeouts with an improved retry mechanism
* PWX-4665 - Do not allow drive add to a pool if a rebalance operation is already in progress
* PWX-4691 - Do not allow snapshots on down nodes or if the node is in maintenance mode
* PWX-4397 - Set the correct zone information for all replica-sets
* PWX-4375 - Add `pxctl upgrade` support for OCI containers
* PWX-4733 - Remove Swarm Node ID check dependencies for Portworx bring up
* PWX-4484 - Limit replication factor increases to a limit of three at a time within a cluster and one per node
* PWX-4090 - Reserve space in each pool to handle rebalance operations
* PWX-4544 - Handle ./aquasec file during OCI-Install so Portworx can be installed in environments with aquasec
* PWX-4497 - Enable minio to mount shared volumes
* PWX-4551 - Improve `pxctl volume inspect` to show pools on which volumes are allocated, replica nodes and replication add
* PWX-4884 - Prevent replication factor increases if all the nodes in the cluster are not running 1.3.0
* PWX-4504 - Show all the volumes present on a node in CLI with a `--node` option
* PWX-4824 - `pxctl volume inspect` doesn’t show replication set information properly when one node is out of quorum
* PWX-4784 - Support SELinux in 4.12.x kernels and above by setting SELinux context correctly
* PWX-4812 - Handle Kernel upgrades correctly
* PWX-4814 - Synchronize snapshot operations per node
* PWX-4471 - Enhancements to OCI Mount propagation to automount relevant scheduler dirs
* PWX-4721 - When a large number of volumes are cloud snapped at the same time, Portworx container hits a panic
* PWX-4789 - Handle cloudsnaps errors when the schedule has been moved or deleted
* PWX-4709 - Support for adding CloudDrive \(EBS volume\) to an existing node in a cluster
* PWX-4777 - Fix issues with `pxctl volume inspect` on shared volumes hanging when a large number of volume inspects are done
* PWX-4525 - `pxctl status` shows an invalid cluster summary in some nodes when performing an upgrade from 1.2 to 1.3
* PWX-3071 - Provide the ability to force detach a remotely mounted Portworx volume from a single node when the node is down
* PWX-4772 - Handle storage full conditions more gracefully when the backing store for a Portworx volume gets full
* PWX-4757 - Improve Portworx initialization during boot to handle out of quorum volumes gracefully.
* PWX-4747 - Improve a simultaneous large number of volume creates and volume attach/detach in multiple nodes
* PWX-4467 - Fix hangs when successive volume inspects come to the same volume with cloudsnap in progress
* PWX-4420 - Fix race between POD delete and volume unmounts
* PWX-4206 - Under certain conditions, creating a snap using Kubernetes PVC creates a new volume instead of a snapshot
* PWX-4207 - Fix nil pointer dereferences when creating snapshots via Kubernetes

### Errata

* PWX-3982 After putting a node into maintenance mode, adding drives, and then running “pxctl service m –exit”, the message “Maintenance operation is in progress, cancel the operation or wait for completion” doesn’t specify which operation hasn’t completed. Workaround: Use pxctl to query the status of all three drive operations \(add, replace, rebalance\). pxctl then reports which drive operations are in progress and allows exiting from maintenance mode if all maintenance operations are completed.
* PWX-4016 When running under Kubernetes, adding a node label for a scheduled cloudsnap fails with the error “Failed to update k8s node”. A node label isn’t needed for cloudsnaps because they are read-only and used only for backup to the cloud.
* PWX-4021 In case of a failure while a read-only snapshot create operation is in progress, Portworx might fail to come back up. This can happen if the failure coincides with snapshot creation’s file system freeze step, which is required to fence incoming IOs during the operation. To recover from this issue, reboot the node.
* PWX-4027 Canceling a service drive replace operation fails with the message “Replace cancel failed - Not in progress”. However, if you try to exit maintenance mode, the status message indicates that a maintenance operation is in progress. Workaround: Wait for the drive to replace operation to finish. The replace operation might be in a state where it can’t be canceled. Cancel operations are performed when possible.
* PWX-4039 When running Ubuntu on Azure, an XFS volume format fails. Do not use XFS volumes when running Ubuntu on Azure.
* PWX-4043 When a Portworx POD gets deleted in Kubernetes, no alerts are generated to indicate the POD deletion via kubectl.
* PWX-4050 For a Portworx cluster that’s about 100 nodes or greater: If the entire cluster goes down with all the nodes offline, as nodes come online a few nodes get restarted because they are marked offline. A short while after, the system converges and the entire cluster becomes operational. No user intervention required.
* Key Management with AWS KMS doesn’t work anymore because of API changes in the AWS side. Will be fixed in an upcoming release. Refer to this link for additional details. https://github.com/aws/aws-cli/issues/1043
* When shared volumes are configured with io\_profile=cms, it results in the px-ns process restarting occasionally.

## 1.2.23

April 20, 2018

This is a minor update that fixes a panic seen in some Kubernetes environments when the user upgraded from an older version of Portworx to 1.2.22

PWX-5107 - Check if node spec is present before adding the node for volume state change events

## 1.2.22

February 28, 2018

### Key Features and Enhancements

* Support SELinux enable in kernels 4.12.x and above
* Support automatic kernel upgrades. If you expect your environment to upgrade kernels automatically, {{<companyName>}} recommends upgrading to 1.2.22.0

## 1.2.20.0

February 15, 2018

* Minor update to enhance write performance for remote mounts with shared volumes
* 4.15.3 Linux kernel support

### Errata \(Errata remains the same from 1.2.11.0 release\)

* PWX-3982 After putting a node into maintenance mode, adding drives, and then running “pxctl service m –exit”, the message “Maintenance operation is in progress, cancel the operation or wait for completion” doesn’t specify which operation hasn’t completed. Workaround: Use pxctl to query the status of all three drive operations \(add, replace, rebalance\). pxctl then reports which drive operations are in progress and allows exiting from maintenance mode if all maintenance operations are completed.
* PWX-4014 The pxctl cloudsnap schedule command creates multiple backups for the scheduled time. This issue has no functional impact and will be resolved in the upcoming release.
* PWX-4016 When running under Kubernetes, adding a node label for a scheduled cloudsnap fails with the error “Failed to update k8s node”. A node label isn’t needed for cloudsnaps because they are read-only and used only for backup to the cloud.
* PWX-4017 An incremental cloudsnap backup command fails with the message “Failed to open snap for backup”. Logs indicate that the backup wasn’t found on at least on one of the nodes where the volume was provisioned. Workaround: Trigger another backup manually on the nodes that failed.
* PWX-4021 In case of a failure while a read-only snapshot create operation is in progress, Portworx might fail to come back up. This can happen if the failure coincides with snapshot creation’s file system freeze step, which is required to fence incoming IOs during the operation. To recover from this issue, reboot the node.
* PWX-4027 Canceling a service drive replace operation fails with the message “Replace cancel failed - Not in progress”. However, if you try to exit maintenance mode, the status message indicates that a maintenance operation is in progress. Workaround: Wait for the drive to replace operation to finish. The replace operation might be in a state where it can’t be canceled. Cancel operations are performed when possible.
* PWX-4039 When running Ubuntu on Azure, an XFS volume format fails. Do not use XFS volumes when running Ubuntu on Azure.
* PWX-4043 When a Portworx POD gets deleted in Kubernetes, no alerts are generated to indicate the POD deletion via kubectl.
* PWX-4050 For a Portworx cluster that’s about 100 nodes or greater: If the entire cluster goes down with all the nodes offline, as nodes come online a few nodes get restarted because they are marked offline. A short while after, the system converges and the entire cluster becomes operational. No user intervention required.
* Key Management with AWS KMS doesn’t work anymore because of API changes in the AWS side. Will be fixed in an upcoming release. Refer to this link for additional details. https://github.com/aws/aws-cli/issues/1043
* PWX-4721 - When cloud-snap is performed on a large number of volumes, it results in a Portworx container restart. A workaround is to run cloudsnaps on up to 10 volumes concurrently.

## 1.2.18.0

February 13, 2018

### Key Features and Enhancements

* Improve file import and untar performance when shared volumes are used by Wordpress and tune for WordPress plugin behavior

### Errata \(Errata remains the same from 1.2.11.0 release\)

* PWX-3982 After putting a node into maintenance mode, adding drives, and then running “pxctl service m –exit”, the message “Maintenance operation is in progress, cancel the operation or wait for completion” doesn’t specify which operation hasn’t completed. Workaround: Use pxctl to query the status of all three drive operations \(add, replace, rebalance\). pxctl then reports which drive operations are in progress and allows exiting from maintenance mode if all maintenance operations are completed.
* PWX-4014 The pxctl cloudsnap schedule command creates multiple backups for the scheduled time. This issue has no functional impact and will be resolved in the upcoming release.
* PWX-4016 When running under Kubernetes, adding a node label for a scheduled cloudsnap fails with the error “Failed to update k8s node”. A node label isn’t needed for cloudsnaps because they are read-only and used only for backup to the cloud.
* PWX-4017 An incremental cloudsnap backup command fails with the message “Failed to open snap for backup”. Logs indicate that the backup wasn’t found on at least on one of the nodes where the volume was provisioned. Workaround: Trigger another backup manually on the nodes that failed.
* PWX-4021 In case of a failure while a read-only snapshot create operation is in progress, Portworx might fail to come back up. This can happen if the failure coincides with snapshot creation’s file system freeze step, which is required to fence incoming IOs during the operation. To recover from this issue, reboot the node.
* PWX-4027 Canceling a service drive replace operation fails with the message “Replace cancel failed - Not in progress”. However, if you try to exit maintenance mode, the status message indicates that a maintenance operation is in progress. Workaround: Wait for the drive to replace operation to finish. The replace operation might be in a state where it can’t be canceled. Cancel operations are performed when possible.
* PWX-4039 When running Ubuntu on Azure, an XFS volume format fails. Do not use XFS volumes when running Ubuntu on Azure.
* PWX-4043 When a Portworx POD gets deleted in Kubernetes, no alerts are generated to indicate the POD deletion via kubectl.
* PWX-4050 For a Portworx cluster that’s about 100 nodes or greater: If the entire cluster goes down with all the nodes offline, as nodes come online a few nodes get restarted because they are marked offline. A short while after, the system converges and the entire cluster becomes operational. No user intervention required.
* Key Management with AWS KMS doesn’t work anymore because of API changes in the AWS side. Will be fixed in an upcoming release. Refer to this link for additional details. https://github.com/aws/aws-cli/issues/1043

## 1.2.16.2

March 19, 2018

* This is a minor update that fixes volume size not updating whenever the content of the encrypted volume is deleted

## 1.2.16.1

March 2, 2018

This is a minor update which adds a new flag to limit or disable the generation of core files \(`-e PXCORESIZE=<size>`\). A value of 0 will disable cores

## 1.2.16.0

February 5, 2018

This is a minor update with performance enhancements for shared volumes to support a large number of directories and files.

### Key Fixes

* Shared volume access latency improvements when managing filesystems with a large number of directories and files

### Errata \(Errata remains the same from 1.2.11.0 release\)

* PWX-3982 After putting a node into maintenance mode, adding drives, and then running “pxctl service m –exit”, the message “Maintenance operation is in progress, cancel the operation or wait for completion” doesn’t specify which operation hasn’t completed. Workaround: Use pxctl to query the status of all three drive operations \(add, replace, rebalance\). pxctl then reports which drive operations are in progress and allows exiting from maintenance mode if all maintenance operations are completed.
* PWX-4014 The pxctl cloudsnap schedule command creates multiple backups for the scheduled time. This issue has no functional impact and will be resolved in the upcoming release.
* PWX-4016 When running under Kubernetes, adding a node label for a scheduled cloudsnap fails with the error “Failed to update k8s node”. A node label isn’t needed for cloudsnaps because they are read-only and used only for backup to the cloud.
* PWX-4017 An incremental cloudsnap backup command fails with the message “Failed to open snap for backup”. Logs indicate that the backup wasn’t found on at least on one of the nodes where the volume was provisioned. Workaround: Trigger another backup manually on the nodes that failed.
* PWX-4021 In case of a failure while a read-only snapshot create operation is in progress, Portworx might fail to come back up. This can happen if the failure coincides with snapshot creation’s file system freeze step, which is required to fence incoming IOs during the operation. To recover from this issue, reboot the node.
* PWX-4027 Canceling a service drive replace operation fails with the message “Replace cancel failed - Not in progress”. However, if you try to exit maintenance mode, the status message indicates that a maintenance operation is in progress. Workaround: Wait for the drive to replace operation to finish. The replace operation might be in a state where it can’t be canceled. Cancel operations are performed when possible.
* PWX-4039 When running Ubuntu on Azure, an XFS volume format fails. Do not use XFS volumes when running Ubuntu on Azure.
* PWX-4043 When a Portworx POD gets deleted in Kubernetes, no alerts are generated to indicate the POD deletion via kubectl.
* PWX-4050 For a Portworx cluster that’s about 100 nodes or greater: If the entire cluster goes down with all the nodes offline, as nodes come online a few nodes get restarted because they are marked offline. A short while after, the system converges and the entire cluster becomes operational. No user intervention required.

## 1.2.14

January 17, 2018

This is a minor update to support the older Linux kernel versions \(4.4.0.x\) that ships with Ubuntu distributions

## 1.2.12.1

January 8, 2018

This is a minor update to support Openshift with SELinux enabled as well as verify with SPECTRE/Meltdown kernel patches

* Verified with the latest kernel patches for SPECTRE/Meltdown issue for all major Linux distros

## 1.2.12.0

December 22, 2017

This is a minor update to enhance metadata performance on a shared namespace volume.

### Key Fixes

* Readdir performance for directories with a large number of files \(greater 128K file count in a single directory\)
* Portworx running on AWS AutoScalingGroup now handles existing devices attached with names such as `/dev/xvdcw` which have an extra letter at the end.
* Occasionally, containers that use shared volumes could get a “transport end point disconnected” error when Portworx restarts. This has been resolved.
* Fixed an issue where Portworx failed to resolve Kubernetes services by their DNS names if the user sets the Portworx DaemonSet DNS Policy as `ClusterFirstWithHostNet`.
* PWX- 4078 When Portworx runs in 100s of nodes, a few nodes show high memory usage.

## 1.2.11.10

December 19, 2017

This is a minor update to address an issue with installing a reboot service while upgrading a runC container.

### Key Fixes

* When upgrading a runC container, the new version will correctly install a reboot service. A reboot service \(systemd service\) is needed to reduce the wait time before a Portworx device returns with a timeout when the Portworx service is down. Without this reboot service, a node can take 10 minutes to reboot.

## 1.2.11.9

December 18, 2017

### Key Fixes

Pass volume name as part of the metrics endpoint so Prometheus/Grafana can display with volume name

* Add current ha level of the volume and io\_priority of the volumes to the metrics endpoint
* Abort all pending I/Os the pxd device during a reboot so speed up reboots
* Move the px-ns internal port from 7000 to 9013
* Remove the unnecessary warning string “Data is not local to the node”
* Add px\_ prefix to all volume labels

### Errata

* Do not manually unmount a volume by using Linux `umount` command for shared volume mounts. This errata applies to the previous versions of Portworx as well.

## 1.2.11.8

December 11, 2017

### Key Fixes

* Fix resync mechanism for read-only snapshots
* Improve log space utilization by removing old log files based on space usage

### Errata

* Do not manually unmount a volume by using Linux `umount` command for shared volume mounts. This errata applies to the previous versions of Portworx as well.

## 1.2.11.7

December 7, 2017

### Key Fixes

* Suppress un-necessary log prints about cache flush
* PWX-4272 Handle remote host shutdowns gracefully for shared volumes. In the past, this could leave stray TCP connections.

### Errata

* Do not manually unmount a volume by using Linux `umount` command for shared volume mounts. This errata applies to the previous versions of Portworx as well.

## 1.2.11.6

November 28, 2017

### Key Fixes

* Provide the capability to drop system cache on-demand \(for select workloads and large memory system\) and turn it off by default

## 1.2.11.5

November 22, 2017

### Key Features and Enhancements

* PWX-4178 Perform snapshots in kubernetes via [annotations](/operations/operate-kubernetes/storage-operations/create-snapshots/on-demand/snaps-annotations)

## 1.2.11.4

November 20, 2017

### Key Features and Enhancements

* {{< pxEnterprise >}} container is now available in [OCI Format](/install-portworx/install-with-other/docker/standalone)
* Enhancements for db workloads to handle slow media

### Key Fixes

* PWX-4224 Ignore `sticky` flag when purging old snapshots after a cloudsnap is completed.
* PWX-4220 `pxctl status` shows the first interface IP address instead of the mgmt. IP

## 1.2.11.3

November 16, 2017

### Key Fixes

* Shared volume performance improvements
* Do not take an inline snap in Kubernetes when no valid candidate pvcs are found

### 1.2.11.2

November 11, 2017

### Key Fixes

* Increase file descriptors to support a large number of shared volumes

## 1.2.11.1

November 7, 2017

### Key Fixes

* Fix file descriptors not being released after reporting containers attached to a shared volume

## 1.2.11

October 31, 2017

### Key Features and Enhancements

* You can now update volume labels. The pxctl volume update command has a new option, –label pairs. Specify a list of comma-separated name=value pairs. For example, if the current labels are x1=v1,x2=v2:

  The option “–labels x1=v4” results in the labels x1=v4,x2=v2.

  The option “–labels x1=” results in the labels x2=v2 \(removes a label\).

* Improvements to alerts:
  * Additional alerts indicate the cluster status in much more finer detail. This document has more details on all the alerts posted by Portworx: [Here](/operations/operate-other/monitoring/alerting)
  * Rate limiting for alerts so that an alert isn’t repeatedly posted within a short timeframe.
* You can now update the io\_profile field by using the `pxctl volume update` command so the parameter can be enabled for existing volumes.

### Key Fixes

* PWX-3146 Portworx module dependencies fail to load for openSUSE Leap 42.2, Kernel 4.4.57-18.3-default.
* PWX-3362 If a node is in maintenance mode because of disk errors, the node isn’t switched to a storage-less node. As a result, other resources on the node \(such as CPU and memory\) aren’t usable.
* PWX-3448 When Portworx statistics are exported, they include the volume ID instead of the volume name.
* PWX-3472 When snapshots are triggered on a large number of volumes at the same time, the snap operation fails.
* PWX-3528 Volume create option parsing isn’t unified across Kubernetes, Docker, and pxctl.
* PWX-3544 Improvements to Portworx diagnostics - REST API to retrieve and upload diagnostics for a node or cluster. Diagnostics run using the REST API includes vmstat output and the output of pxctl cluster list and pxctl --json volume list. The diagnostics also include netstat -s before the node went down.
* PWX-3558 px-storage dumps core while running an HA increase on multiple volumes during stress.
* PWX-3577 When Portworx is running in a container environment, it should allow mounts on only those directories which are bind-mounted. Otherwise, Portworx hangs during a docker stop.
* PWX-3585 If Portworx stops before a container that’s using its volume stops, the container might get stuck in the D state \(I/O in kernel\). As a result, ‘systemctl stop docker’ takes 10 minutes as does system shutdown. The default PXD\_TIMEOUT to error out IOs is 10 minutes, but should be configurable.
* PWX-3591 Storage isn’t rebalanced after a drive add operation and before exiting maintenance mode.
* PWX-3600 Volume HA update operations on snapshots cannot be canceled.
* PWX-3602 Removing a node from a cluster fails with the message “Could not find any volumes that match ID\(s\)”.
* PWX-3606 Portworx metrics now include the following: Disk read and write latency stats, volume read and write latency stats, and per-process stats for CPU and virtual/resident memory.
* PWX-3612 When creating or updating a volume, disallow the ability to set both the “shared” and “scale” options.
* PWX-3614 A volume inspect returns the wrong error message when one node in the cluster is down: Could not find any volumes that match ID\(s\).
* PWX-3620 The volume inspect command doesn’t show the replication set status, such as whether the replication set has down members or is in a clean or resync state.
* PWX-3632 After a Kubernetes pod terminates and the Portworx volume unmount/cleanup fails, the kubelet logs include “Orphaned pod &lt;name&gt; found, but volume paths are still present on disk.”
* PWX-3648 After all nodes in a cluster go offline: If a node doesn’t restart when the other nodes restart, the other restarting nodes don’t mark that node as offline.
* PWX-3665 The Portworx live core collection hangs sometimes.
* PWX-3666 The pxctl service diags command doesn’t store all diagnostics for all nodes in the same location. All diagnostics should appear in /var/cores.
* PWX-3672 The watch function stops after a large time change, such as 7 hours, on the cluster.
* PWX-3678 The pxctl volume update command interprets the -s option as -shared instead of -size and displays the message “invalid shared flag”.
* PWX-3700 Multiple alerts appear after a drive add succeeds.
* PWX-3701 The alert raised when a node enters maintenance mode specifies the node index instead of the node ID.
* PWX-3704 After backing up a volume that’s in maintenance mode to the cloud, restoring the volume to any online node fails.
* PWX-3709 High CPU usage occurs while detaching a volume with MySQL in Docker Swarm mode.
* PWX-3743 In the service alerts output in the CLI, the Description items aren’t aligned.
* PWX-3746 When a Portworx upgrade requires a node reboot, the message “Upgrade done” shouldn’t print.
* PWX-3747 When a node exits from maintenance mode, it doesn’t generate an alert.
* PWX-3764 The px-runc install command on a core node fails to configure the Portworx OCI service and generates the error “invalid cross-device link”.
* PWX-3777 When running under Kubernetes, pods using a shared volume aren’t available after the volume becomes read-only.
* PWX-3778 After adding a drive to a storage-less node fails: A second attempt succeeds but there is no message that the drive add succeeded.
* PWX-3793 When running in Kubernetes, if an unmount fails for a shared volume with the error “volume not mounted”, the volume is stuck in a terminating state.
* PWX-3817 When running under Kubernetes, a WordPress pod is stuck in terminating for almost ten minutes.
* PWX-3820 When running Portworx as a Docker V2 plugin: After a service create –replicas command, a volume is mounted locally on a MySQL container instead of a Portworx container. The Swarm service fails with the error “404 Failed to locate volume: Cannot locate volume”. To avoid this issue, you can now specify the volume-driver with the service create command.
* PWX-3825 When a node is in storage down state because the pool is out of capacity: A drive add fails with the error “Drive add start failed. drive size &lt;size&gt; too big” during an attempt to add the same size disk.
* PWX-3829 Container status in the Portworx Lighthouse GUI isn’t updated properly from Portworx nodes.
* PWX-3843 Portworx stats include metrics for utilized and available bytes, but not for total bytes \(px\_cluster\_disk\_total\_bytes\). As a result, alerts can’t be generated in Prometheus for storage utilization.
* PWX-3844 When you add a snapshot schedule to a volume, the alert type is “Snapshot Interval update failure” instead of “Snapshot interval update success”.
* PWX-3850 If the allocated io\_priority differs from the requested io\_priority, no associated alert is generated.
* PWX-3851 When two Postgres pods attempted to use the same volume, one of the Postgres pods mounted a local volume instead of a Portworx volume.
* PWX-3859 After adding a volume template to an Auto Scaling Group and Portworx adds tags to the volume: If you stop that cluster and then start a new cluster with the same volume, without removing the tags, a message indicates that the cluster is already initialized. The message should indicate that it failed to attach template volumes because the tag is already used. You can then manually remove the tags from the stopped cluster.
* PWX-3862 A volume is stuck in the detaching state indefinitely due to an issue in etcd.
* PWX-3867 When running under Kubernetes, a pod using namespace volumes generates the messages “Orphaned pod &lt;pod&gt; found, but volume paths are still present on disk”.
* PWX-3868 A Portworx cluster shows an extra node when running with ASG templates enabled if the AWS API returns an error when the Portworx container is booting up.
* PWX-3871 Added support for dot and hyphen in source and destination names in Kubernetes inline spec for snapshots.
* PWX-3873 When running under Kubernetes, a volume detach fails on a regular volume, with the message “Failed to detach volume: Failed with status -16”, and px-storage dumps core.
* PWX-3875 After volume unmount and mount commands are issued in quick succession, sometimes the volume mount fails.
* PWX-3878 When running under Kubernetes, a Postgres pod gets stuck in a terminating state during when the POD gets deleted.
* PWX-3879 During volume creation on Kubernetes, node labels aren’t applied on Kubernetes nodes.
* PWX-3888 An HA increase doesn’t use the node value specified in the command if the node is from a different region.
* PWX-3895 The pxctl volume list command shows a volume but volume inspect cannot find it.
* PWX-3902 If a Portworx container is started with the API\_SERVER pointing to Lighthouse and etcd servers are also provided, the Portworx container doesn’t send statistics to Lighthouse.
* PWX-3906 Orphaned pod volume directories can remain in a Kubernetes cluster after an unmount.
* PWX-3912 During a container umount, namespace volumes might show the error “Device or resource busy”.
* PWX-3916 Portworx rack information isn’t updated when labels are applied to a Kubernetes node.
* PWX-3933 The size of a volume created by using a REST API call isn’t rounded to the 4K boundary.
* PWX-3935 Lighthouse doesn’t show container information when Portworx is run as a Docker V2 plugin.
* PWX-3936 A volume create doesn’t ignore storage-less nodes in a cluster and thus fails, because it doesn’t allocate the storage to available nodes.
* PWX-3946 On a node where a cloudsnap schedule is configured: If the node gets decommissioned, the schedule isn’t configured for the new replica set.
* PWX-3947 Simultaneous mount and unmount likely causes a race in teardown and setup.
* PWX-3968 If Portworx can’t find a volume template in an Auto Scaling Group, it dumps core and keeps restarting.
* PWX-3971 Portworx doesn’t install on an Azure Ubuntu 14 Distro with the 3.13.0-32-generic kernel.
* PWX-3972 When you start a multi-node, multi-zone Auto Scaling Group with a max-count specified, Portworx doesn’t start on all nodes.
* PWX-3974 When running under Kubernetes, a WordPress app writes data to the local filesystem after a shared volume remount failure \(due to RPC timeout errors\) during node start.
* PWX-3997 When running under Kubernetes, deleting Wordpress pods results in orphaned directories.
* PWX-4000 A drive add or replace fails when Portworx is in storage full/pool offline state.
* PWX-4012 When using shared volumes: During a WordPress plugin installation, the WordPress pod prompts for FTP site permissions. Portworx now passes the correct GID and UUID to WordPress.
* PWX-4049 Adding and removing Kubernetes node labels can fail during node updates.
* PWX-4051 Previous versions of Portworx logged too many “Etcd did not return any transaction responses” messages. That error is now rate-limited to log only a few times.
* PWX-4083 When volume is in a down state due to a create failure but is still attached without a shared volume export, the detach fails with the error “Mountpath is not mounted”.
* PWX-4085 When running under Kubernetes, too many instances of this message get generated: “Kubernetes node watch channel closed. Restarting the watch..”
* PWX-4131 Specifying -a or -A for providing disks to Portworx needs to handle mpath & raid drives/partitions as well

  **Errata**

* PWX-3982 After putting a node into maintenance mode, adding drives, and then running “pxctl service m –exit”, the message “Maintenance operation is in progress, cancel the operation or wait for completion” doesn’t specify which operation hasn’t completed. Workaround: Use pxctl to query the status of all three drive operations \(add, replace, rebalance\). pxctl then reports which drive operations are in progress and allows exiting from maintenance mode if all maintenance operations are completed.
* PWX-4014 The pxctl cloudsnap schedule command creates multiple backups for the scheduled time. This issue has no functional impact and will be resolved in the upcoming release.
* PWX-4016 When running under Kubernetes, adding a node label for a scheduled cloudsnap fails with the error “Failed to update k8s node”. A node label isn’t needed for cloudsnaps because they are read-only and used only for backup to the cloud.
* PWX-4017 An incremental cloudsnap backup command fails with the message “Failed to open snap for backup”. Logs indicate that the backup wasn’t found on at least on one of the nodes where the volume was provisioned. Workaround: Trigger another backup manually on the nodes that failed.
* PWX-4021 In case of a failure while a read-only snapshot create operation is in progress, Portworx might fail to come back up. This can happen if the failure coincides with snapshot creation’s file system freeze step, which is required to fence incoming IOs during the operation. To recover from this issue, reboot the node.
* PWX-4027 Canceling a service drive replace operation fails with the message “Replace cancel failed - Not in progress”. However, if you try to exit maintenance mode, the status message indicates that a maintenance operation is in progress. Workaround: Wait for the drive to replace operation to finish. The replace operation might be in a state where it can’t be canceled. Cancel operations are performed when possible.
* PWX-4039 When running Ubuntu on Azure, an XFS volume format fails. Do not use XFS volumes when running Ubuntu on Azure.
* PWX-4043 When a Portworx POD gets deleted in Kubernetes, no alerts are generated to indicate the POD deletion via kubectl.
* PWX-4050 For a Portworx cluster that’s about 100 nodes or greater: If the entire cluster goes down with all the nodes offline, as nodes come online a few nodes get restarted because they are marked offline. A short while after, the system converges and the entire cluster becomes operational. No user intervention required.

## 1.2.10.2

October 6, 2017

### Key Fixes

* Fix boot issues with Amazon Linux
* Fix issues with shared volume mount and unmount with multiple containers with kubernetes

## 1.2.10

September 18, 2017

### Key Fixes

* Fix issue when a node running Portworx goes down, it never gets marked down in the kvdb by other nodes.
* Fix issue when a container in Lighthouse UI always shows as running even after it has exited
* Auto re-attach containers mounting shared volumes when Portworx container is restarted.
* Add Linux immutable \(CAP\_LINUX\_IMMUTABLE\) when Portworx is running as Docker V2 Plugin
* Set autocache parameter for shared volumes
* On volume mount, make the path read-only if an unmount comes in if the POD gets deleted or Portworx is restarted during POD creation. On unmount,  delete the mount path.
* Remove the volume quorum check during volume mounts so the mount can be retried until the quorum is achieved
* Allow snapshot volume source to be provided as another Portworx volume ID and Snapshot ID
* Allow inline snapshot creation in Portworx Kubernetes volume driver using the Portworx Kubernetes volume spec
* Post log messages indicating when logging URL is changed
* Handle volume delete requests gracefully when Portworx container is starting up
* Handle service account access when Portworx is running as a container instead of a daemonset when running under kubernetes
* Implement a global lock for kubernetes filter such that all cluster-wide Kubernetes filter operations are coordinated through the lock
* Improvements in unmounting/detaching handling in kubernetes to handle different POD clean up behaviors for deployments and statefulsets

### Errata

* If two containers using the same shared volume are run in the same node using docker, when one container exits, the container’s connection to the volume will get disrupted as well. The workaround is to run containers using shared volume in two different Portworx nodes

## 1.2.9

August 23, 2017

{{<info>}}
**Important:**
If you are upgrading from an older version of Portworx (1.2.8 or older) and have Portworx volumes in the attached state, you will need node reboot after upgrade in order for the new version to take effect properly.
{{</info>}}

### Key Features and Enhancements

* Provide the ability to cancel a replication add or HA increase operation
* Automatically decommission a storage less node in the cluster if it has been offline for longer than 48 hours
* [Kubernetes snapshots driver for {{< pxEnterprise >}}](/operations/operate-kubernetes/storage-operations/create-snapshots)
* Improve Kubernetes mount/unmount handling with POD failovers and moves

### Key Fixes

* Correct mountpath retrieval for encrypted volumes
* Fix cleanup path maintenance mode exit issue and clean up alerts
* Fix S3 provider for compatibility issues with legacy object storage providers not supporting ListObjectsV2 API correctly.
* Add more cloudsnap related alerts to indicate cloudsnap status and any cloudsnap operation failures.
* Fix config.json for Docker Plugin installs
* Read topology parameters on Portworx restart so RACK topology information is read correctly on restarts
* Retain environment variables when Portworx is upgraded via `pxctl upgrade` command
* Improve handling for encrypted scale volumes

### Errata

* When {{< pxEnterprise >}} is run on a large number of nodes, there is potential memory leak and a few nodes show high memory usage. This issue is resolved in 1.2.12.0 onwards. The workaround is to restart the {{< pxEnterprise >}} container

## 1.2.8

June 27, 2017

### Key Features and Enhancements

* License Tiers for {{< pxEnterprise >}}

## 1.2.5 Release notes

June 16, 2017

### Key Features and Enhancements

* Increase volume limit to 16K volumes

### Key Fixes

* Fix issues with volume CLI hitting a panic when used the underlying devices are from LVM devices
* Fix Portworx bootstrap issues with pre-existing snapshot schedules
* Remove alerts posted when volumes are mounted and unmounted
* Remove duplicate updates to kvdb

## 1.2.4

June 8, 2017

#### Key Features and Enhancements

* Support for –racks and –zones option when creating replicated volumes
* Improved replication node add speeds
* Node labels and scheduler convergence for docker swarm
* Linux Kernel 4.11 support
* Unique Cluster-specific bucket for each cluster for cloudsnaps
* Load balanced cloudsnap backups for replicated Portworx volumes
* One-time backup schedules for Cloudsnap
* Removed the requirement to have /etc/pwx/kubernetes.yaml in all Kubernetes nodes

### Key Fixes

* `pxctl cloudsnap credentials` command has been moved under `pxctl credentials`
* Docker inline volume creation support for setting volume aggregation level
* –nodes support for docker inline volume spec
* Volume attach issues after a node restart when container attaching to a volume failed
* Portworx displays issues in Prometheus
* Cloudsnap scheduler display issues where the existing schedules were not seen by some users.
* Removed snapshots from being counted into to total volume count
* Removed non-Portworx related metrics being pushed to Prometheus
* Added CLI feedback and success/failure alerts for `pxctl volume update` command
* Fixed issues with Cloudsnap backup status updates for container restarts

## 1.2.3

May 30, 2017

### Key Features and Enhancements

No new features in 1.2.3. This is a patch release.

### Key Fixes

* Performance improvements for database workloads

## 1.2.2

May 24, 2017

### Key Features and Enhancements

No new features in 1.2.2. This is a patch release.

### Key Fixes

* Fix device detection in AWS authenticated instances

## 1.2.1

May 9, 2017

### Key Features and Enhancements

No new features in 1.2.1. This is a patch release.

### Key Fixes

* Fix issues with pod failovers with encrypted volumes
* Improve performance with remote volume mounts
* Add compatibility for Linux 4.10+ kernels

## 1.2.0

April 27, 2017

### Key Features and Enhancements

* [AWS Auto-scaling integration with Portworx](/install-portworx/cloud/aws/aws-asg) managing EBS volumes for EC2 instances in AWS ASG
* [Multi-cloud Backup and Restore](/reference/cli/cloud-snaps) of Portworx volumes
* [Encrypted Volumes](/reference/cli/encrypted-volumes) with Data-at-rest and Data-in-flight encryption
* Docker V2 Plugin Support
* [Prometheus Integeration](/operations/operate-other/monitoring/prometheus)
* [Hashicorp Vault](/operations/key-management/vault), [AWS KMS integration](/install-portworx/cloud/aws) and Docker Secrets Integration
* [Dynamically resize](/reference/cli/updating-volumes) Portworx volumes with no application downtime
* Improved the security of the Portworx container

### Key Fixes

* Issues with volume auto-attach
* Improved network diagnostics on Portworx container start
* Added an alert when volume state transitions to read-only due to loss of quorum
* Display multiple attached hosts on shared volumes
* Improve shared volume container attach when the volume is in resync state
* Allow pxctl to run as a normal user
* Improved pxctl help text for commands like pxctl service
