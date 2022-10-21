---
title: Install Portworx for PX-Fast
linkTitle: PX-Fast
description: Installation instructions for Portworx Enterprise on Kubernetes with PX-StoreV2 datastore and PX-Fast
keywords: portworx, kurbernetes, PX-StoreV2, fastpath
weight: 600
disableprevnext: true
---

PX-Fast is a Portworx feature that enables an accelerated IO path for the volumes that meet certain prerequisites. It is optimized for workloads requiring consistent low latencies. PX-Fast is built on top of PX-StoreV2 datastore, and requires a special license for the functionality to be active. Contact [Portworx support](/support) team for obtaining this license. 

PX-StoreV2 is a Portworx datastore optimized for supporting IO intensive workloads for configurations utilizing high performance NVMe class devices. It's the platform that enables PX-Fast. PX-StoreV2 is targeted for workloads requiring high data ingestion rates with consistent latencies.

## Prerequisites

You must have a Kubernetes cluster deployed on infrastructure that meets the following minimum requirements for Portworx:

* Linux kernel version: 4.20 or newer is the minimum required version. 5.0 or newer is recommended, with the following packages:
    * **Rhel**: device-mapper mdadm lvm2 device-mapper-persistent-data augeas
    * **Debian**: dmsetup mdadm lvm2 thin-provisioning-tools augeas-tools
    * **Suse**: dmsetup mdadm lvm2 device-mapper-persistent-data augeas

    {{<info>}}**NOTE:** During installation, Portworx will automatically try to pull the required packages from distribution specific repositories. This is a mandatory requirement and installation will fail if this prerequisite is not met.{{</info>}}

* A minimum of 64 GB system metadata device on each node where you want to deploy Portworx.
* An NVMe or SSD drive (cloud drives are not supported).


## Install Portworx

Follow the instructions in this section to deploy Portworx with the PX-StoreV2 datastore.

{{<info>}}**NOTE:**

* PX-StoreV2 datastore installation is supported only with a fresh on-premises installation.

* Upgrading from a previous Portworx version to deploy PX-StoreV2 datastore is not supported.

{{</info>}}

### Generate the specs

To install Portworx with Kubernetes, you must first generate Kubernetes manifests that you will deploy in your cluster:

1. Navigate to [PX-Central](https://central.portworx.com/specGen/wizard) to log in, or create an account.


2. Select **{{< pxEnterprise >}}** from the Product Catalog page.

3. On the Product Line page, choose any option depending on which license you intend to use, then click **Continue** to start the spec generator.

4. On the Basic page, choose at least version 2.12 version from the **Portworx version** dropdown. In the **Namespace** field, specify an existing namespace where you will be deploying Portworx and click **Next**. 

5. On the Storage page, select **On Premises** as your environment and select the type of **OnPrem storage**. Choose the **PX-StoreV2** option and provide your meta device path in the **Metadata Path** textbox. Select **Skip KVDB device** option and then click **Next**:

    ![PX-StoreV2 screen](/img/px-store-v2-install.png)

6. Provide your network options and click **Next**.


7. On the Customize page, choose **None** for the **Are you running on either of these?** option, and click **Finish** to generate the specs. 

Once you've generated your specs, you're ready to apply them and deploy Portworx. You also can save your specs on PX-Central for future reference. 

### Apply specs

Apply the specs using the following commands:

1. Deploy the Operator:

    ```
    kubectl apply -f 'https://install.portworx.com/<version-number>?comp=pxoperator'
    ```
    ```output
    serviceaccount/portworx-operator created
    podsecuritypolicy.policy/px-operator created
    clusterrole.rbac.authorization.k8s.io/portworx-operator created
    clusterrolebinding.rbac.authorization.k8s.io/portworx-operator created
    deployment.apps/portworx-operator created
    ```

2. Deploy the StorageCluster:

    ```
    kubectl apply -f 'https://install.portworx.com/<version-number>?operator=true&mc=false&kbver=&b=true&c=px-cluster-0d8dad46-f9fd-4945-b4ac-8dfd338e915b&stork=true&csi=true&mon=true&tel=false&st=k8s&promop=true'
    ```
    ```output
    storagecluster.core.libopenstorage.org/px-cluster-378d7ae1-f4ca-4f92-914d-fab038f0bbe6 created
    ```

{{<info>}}
**NOTE:** 

* In your output, the image pulled will differ based on your chosen Portworx license type and version.
* For {{< pxEnterprise >}}, the default license activated on the cluster is a 30 day trial that you can convert to a SaaS-based model or a generic fixed license.
* For {{< pxEssentials >}}, your cluster must have internet connectivity so that it can send usage information every 24 hours to renew the license on the cluster. You can convert a {{< pxEssentials >}} license to either a fixed license or SaaS-based license.
{{</info>}}

## Verify your Portworx installation 

Once you've installed Portworx, you can perform the following tasks to verify that Portworx is correctly installed and using the PX-StoreV2 datastore.

### Verify if all pods are running

Enter the following command to list and filter the results for Portworx pods and specify the namespace where you have deployed Portworx:

```text 
kubectl get pods -n <px-namespace> -o wide | grep -e portworx -e px
```

```output
NAME                                                    READY   STATUS    RESTARTS         AGE     IP              NODE                         NOMINATED NODE   READINESS GATES
portworx-api-8scq2                                      1/1     Running   1 (90m ago)      5h1m    xx.xx.xxx.xxx   username-vms-silver-sight-0   <none>           <none>
portworx-api-f24b9                                      1/1     Running   1 (108m ago)     5h1m    xx.xx.xxx.xxx   username-vms-silver-sight-3   <none>           <none>
portworx-api-f95z5                                      1/1     Running   1 (90m ago)      5h1m    xx.xx.xxx.xxx   username-vms-silver-sight-2   <none>           <none>
portworx-kvdb-558g5                                     1/1     Running   0                3m46s   xx.xx.xxx.xxx   username-vms-silver-sight-2   <none>           <none>
portworx-kvdb-9tfjd                                     1/1     Running   0                2m57s   xx.xx.xxx.xxx   username-vms-silver-sight-0   <none>           <none>
portworx-kvdb-cjcxg                                     1/1     Running   0                3m7s    xx.xx.xxx.xxx   username-vms-silver-sight-3   <none>           <none>
portworx-operator-548b8d4ccc-qgnkc                      1/1     Running   13 (4m26s ago)   5h2m    xx.xx.xxx.xxx   username-vms-silver-sight-0   <none>           <none>
portworx-pvc-controller-ff669698-62ngd                  1/1     Running   1 (108m ago)     5h1m    xx.xx.xxx.xxx   username-vms-silver-sight-3   <none>           <none>
portworx-pvc-controller-ff669698-6b4zj                  1/1     Running   1 (90m ago)      5h1m    xx.xx.xxx.xxx   username-vms-silver-sight-2   <none>           <none>
portworx-pvc-controller-ff669698-pffvl                  1/1     Running   1 (90m ago)      5h1m    xx.xx.xxx.xxx   username-vms-silver-sight-0   <none>           <none>
prometheus-px-prometheus-0                              2/2     Running   2 (90m ago)      5h      xx.xx.xxx.xxx   username-vms-silver-sight-0   <none>           <none>
px-cluster-378d7ae1-f4ca-4f92-914d-fab038f0bbe6-2qsp4   2/2     Running   13 (108m ago)    3h20m   xx.xx.xxx.xxx   username-vms-silver-sight-3   <none>           <none>
px-cluster-378d7ae1-f4ca-4f92-914d-fab038f0bbe6-5vnzv   2/2     Running   16 (90m ago)     3h20m   xx.xx.xxx.xxx   username-vms-silver-sight-0   <none>           <none>
px-cluster-378d7ae1-f4ca-4f92-914d-fab038f0bbe6-lxzd5   2/2     Running   16 (90m ago)     3h20m   xx.xx.xxx.xxx   username-vms-silver-sight-2   <none>           <none>
px-csi-ext-77fbdcdcc9-7hkpm                             4/4     Running   4 (108m ago)     3h19m   xx.xx.xxx.xxx   username-vms-silver-sight-3   <none>           <none>
px-csi-ext-77fbdcdcc9-9ck26                             4/4     Running   4 (90m ago)      3h18m   xx.xx.xxx.xxx   username-vms-silver-sight-0   <none>           <none>
px-csi-ext-77fbdcdcc9-ddmjr                             4/4     Running   14 (90m ago)     3h20m   xx.xx.xxx.xxx   username-vms-silver-sight-2   <none>           <none>
px-prometheus-operator-7d884bc8bc-5sv9r                 1/1     Running   1 (90m ago)      5h1m    xx.xx.xxx.xxx   username-vms-silver-sight-0   <none>           <none>
```

Note the name of one of your `px-cluster` pods. You'll run `pxctl` commands from these pods in following steps. 

### Verify Portworx cluster status

You can find the status of the Portworx cluster by running `pxctl status` commands from a pod. Enter the following `kubectl exec` command, specifying the pod name you retrieved in the previous section:

```text
kubectl exec <px-pod>  -n <px-namespace> -- /opt/pwx/bin/pxctl status
```
```output
Defaulted container "portworx" out of: portworx, csi-node-driver-registrar
Status: PX is operational
Telemetry: Disabled or Unhealthy
Metering: Disabled or Unhealthy
License: Trial (expires in 31 days)
Node ID: 24508311-e2fe-42cd-88f3-bf578f9addc1
	IP: xx.xx.xxx.xxx 
 	Local Storage Pool: 1 pool
	POOL	IO_PRIORITY	RAID_LEVEL	USABLE	USED	STATUS	ZONE	REGION
	0	HIGH		raid0		25 GiB	33 MiB	Online	default	default
	Local Storage Devices: 1 device
	Device	Path		Media Type		Size		Last-Scan
	0:0	/dev/sda	STORAGE_MEDIUM_SSD	32 GiB		10 Oct 22 23:45 UTC
	total			-			32 GiB
	Cache Devices:
	 * No cache devices
	Kvdb Device:
	Device Path	Size
	/dev/sdc	1024 GiB
	 * Internal kvdb on this node is using this dedicated kvdb device to store its data.
	Metadata Device: 
	1	/dev/sdd	STORAGE_MEDIUM_SSD	64 GiB
Cluster Summary
	Cluster ID: px-cluster-378d7ae1-f4ca-4f92-914d-fab038f0bbe6
	Cluster UUID: 482b18b1-2a8b-4f10-b992-5d610fa334bd
	Scheduler: kubernetes
	Nodes: 3 node(s) with storage (3 online)
	IP		ID					SchedulerNodeName		Auth		StorageNode		Used	Capacity	Status	StorageStatus	Version		Kernel				OS
	xx.xx.xxx.xxx	24508311-e2fe-42cd-88f3-bf578f9addc1	username-vms-silver-sight-3	Disabled	Yes(PX-StoreV2)	33 MiB	25 GiB		Online	Up (This node)	2.12.0-28944c8	5.4.217-1.el7.elrepo.x86_64	CentOS Linux 7 (Core)
	xx.xx.xxx.xxx	1e89102f-0510-4df3-aba4-4a1bafeff5bc	username-vms-silver-sight-0	Disabled	Yes(PX-StoreV2)	33 MiB	25 GiB		Online	Up		2.12.0-28944c8	5.4.217-1.el7.elrepo.x86_64	CentOS Linux 7 (Core)
	xx.xx.xxx.xxx	0c99e1f2-9d49-4d53-9315-502e658bc307	username-vms-silver-sight-2	Disabled	Yes(PX-StoreV2)	33 MiB	25 GiB		Online	Up		2.12.0-28944c8	5.4.217-1.el7.elrepo.x86_64	CentOS Linux 7 (Core)
Global Storage Pool
	Total Used    	:  99 MiB
	Total Capacity	:  74 GiB
```

The Portworx status will display `PX is operational`, and the `StorageNode` entries for each node will read `Yes(PX-StoreV2)`.

### Verify Portworx pool status

Run the following command to view the Porworx drive configurations for your pod:

```text
kubectl exec <px-pod>  -n <px-namespace> -- /opt/pwx/bin/pxctl service pool show
```
```output
Defaulted container "portworx" out of: portworx, csi-node-driver-registrar
PX drive configuration:
Pool ID: 0 
	Type:  PX-StoreV2 
	UUID:  58ab2e3f-a22e-492a-ad39-db8abe01d4f0 
	IO Priority:  HIGH 
	Labels:  kubernetes.io/arch=amd64,kubernetes.io/hostname=username-vms-silver-sight-3,kubernetes.io/os=linux,medium=STORAGE_MEDIUM_SSD,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,iopriority=HIGH 
	Size: 25 GiB 
	Status: Online 
	Has metadata:  No 
	Balanced:  Yes 
	Drives:
	0: /dev/sda, Total size 32 GiB, Online
	Cache Drives:
	No Cache drives found in this pool
Metadata  Device:
	1: /dev/sdd, STORAGE_MEDIUM_SSD
```
The `Type: PX-StoreV2` output shows that your pod is using the PX-StoreV2 datastore.

### Verify pxctl cluster provision status

* Find the storage cluster using the following command; the status should show the cluster is `Online`:

    ```text
    kubectl -n <px-namespace> get storagecluster
    ```
    ```output
    NAME                                              CLUSTER UUID                           STATUS   VERSION          AGE
    px-cluster-378d7ae1-f4ca-4f92-914d-fab038f0bbe6   482b18b1-2a8b-4f10-b992-5d610fa334bd   Online   2.12.0-dev-rc1   5h6m
    ```

* Find the storage nodes, whose statuses should show as `Online`:

    ```text
    kubectl -n <px-namespace> get storagenodes
    ```

    ```output
    NAME                          ID                                     STATUS   VERSION          AGE
    username-vms-silver-sight-0   1e89102f-0510-4df3-aba4-4a1bafeff5bc   Online   2.12.0-28944c8   3h25m
    username-vms-silver-sight-2   0c99e1f2-9d49-4d53-9315-502e658bc307   Online   2.12.0-28944c8   3h25m
    username-vms-silver-sight-3   24508311-e2fe-42cd-88f3-bf578f9addc1   Online   2.12.0-28944c8   3h25m
    ```       

* Verify the Portworx cluster provision status. Enter the following `kubectl exec` command, specifying the pod name you retrieved in the previous section:

    ```text
    kubectl exec <px-pod> -n <px-namespace> -- /opt/pwx/bin/pxctl cluster provision-status
    ```

    ```output
    NODE					                NODE STATUS	 POOL						              POOL STATUS  IO_PRIORITY	SIZE	AVAILABLE	USED   PROVISIONED ZONE REGION	RACK
    0c99e1f2-9d49-4d53-9315-502e658bc307	Up		    0 ( 8ec9e6aa-7726-4855-a9f1-f9131bf7ef9d )	Online		HIGH		32 GiB	32 GiB		33 MiB	0 B		default	default	default
    1e89102f-0510-4df3-aba4-4a1bafeff5bc	Up		    0 ( 06fcc73a-7e2f-4365-95ce-434152789beb )	Online		HIGH		32 GiB	32 GiB		33 MiB	0 B		default	default	default
    24508311-e2fe-42cd-88f3-bf578f9addc1	Up		    0 ( 58ab2e3f-a22e-492a-ad39-db8abe01d4f0 )	Online		HIGH		32 GiB	32 GiB		33 MiB	0 B		default	default	default
    ```

## Create PVCs

Once Portwox is installed, you can proceed to [Create PX-Fast PVCs](/operations/operate-kubernetes/storage-operations/create-pvcs/px-fastpath-pvc).

{{<info>}}**NOTE:** The following are not supported with PX-StoreV2/PX-Fast:

* XFS Volumes
* PX-Cache
* Autopilot 
* PDS 
* Portworx Backup/DR
* Cloud/ASG drives
{{</info>}}
