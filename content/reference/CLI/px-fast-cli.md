---
title: PX-Fast volumes using pxctl
linkTitle: PX-Fast volumes
keywords: pxctl, command line, PX-fast, PX-Storev2, PX-StoreV2 
description: Provides pxctl command operations for PX-Fast volumes
weight: 110
disableprevnext: true
---

This section provides instructions for performing operations on PX-Fast volumes using `pxctl`.

## Create a PX-Fast volume

To create a PX-fast volume, run the following command on any of the worker nodes of your cluster:

```text
pxctl volume create --fastpath <px-volume>
```
```output
Volume successfully created: 913654286436616938
```

## Attach a volume

Attach you PX-Fast volume to the host on the same node:

```text
pxctl host attach <px-volume>
```

## List volumes in your cluster 

Run the following command to get the list of volumes within a cluster:

```text
pxctl volume list
```

```output
ID			NAME						SIZE	HA	SHARED	ENCRYPTED	PROXY-VOLUME	IO_PRIORITY	STATUS		SNAP-ENABLED	
1150247763189801500	pvc-d7941878-19c9-43d3-b404-84d9193d55a5	2 GiB	no	no		no		LOW		up - detached		no
79554029234843357	v1						1 GiB	no	no		no		LOW		up - detached		no
909303106655754070	v2						1 GiB	no	no		no		LOW		up - attached on 10.13.160.65	no
```
Note the ID or name of one of the volume; you can use it to view details using the `pxctl` command.

## Get volume details 

Run the following command to get the detailed PX-Fast setting and usage information for a volume:

```text
pxctl volume inspect -e <px-volume>
```
```output
Volume          	 :  909303106655754070
	Name            	 :  v2
	Size            	 :  1.0 GiB
	Format          	 :  ext4
	HA              	 :  1
	IO Priority     	 :  LOW
	Creation time   	 :  Oct 11 01:43:36 UTC 2022
	Shared          	 :  no
	Status          	 :  up
	State           	 :  Attached: 0c99e1f2-9d49-4d53-9315-502e658bc307 (10.13.160.65)
	Last Attached   	 :  Oct 11 01:43:42 UTC 2022
	Device Path     	 :  /dev/pxd/pxd909303106655754070
	Mount Options          	 :  discard
	Reads           	 :  40
	Reads MS        	 :  20
	Bytes Read      	 :  1060864
	Writes          	 :  0
	Writes MS       	 :  0
	Bytes Written   	 :  0
	IOs in progress 	 :  0
	Bytes used      	 :  33 MiB
	Replica sets on nodes:
		Set 0
		  Node 		 : 10.13.160.65 (Pool 8ec9e6aa-7726-4855-a9f1-f9131bf7ef9d )
	Replication Status	 :  Up

	Displaying extended volume state:
	Fastpath preferred	: true
	Fastpath promoted	: true
	Fastpath dirty   	: false
	Fastpath force failover	: false
	Fastpath attached	: FASTPATH_ACTIVE
	Fastpath coordinator	: 0c99e1f2-9d49-4d53-9315-502e658bc307
	Fastpath replicas	: 1
	Fastpath replica property follows:
		Replica	: 0
			On Node		: 0c99e1f2-9d49-4d53-9315-502e658bc307
			Protocol	: FASTPATH_PROTO_LOCAL
			Secure		: true
			Exported	: true
				Target	: /dev/pwx0/909303106655754070
				Source	: /dev/pwx0/909303106655754070
				Type	: Block device
			Imported	: true
				Mapped local device: /dev/pwx0/909303106655754070
```


## Get pool details servicing your volume 

Run the following command to view the details of the pool servicing your volume:

```text 
pxctl service pool show
```
```output
PX drive configuration:
Pool ID: 0 
	Type:  PX-StoreV2 
	UUID:  58ab2e3f-a22e-492a-ad39-db8abe01d4f0 
	IO Priority:  HIGH 
	Labels:  medium=STORAGE_MEDIUM_SSD,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,iopriority=HIGH,kubernetes.io/arch=amd64,kubernetes.io/hostname=username-vms-silver-sight-3,kubernetes.io/os=linux 
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
## Update a volume to enable PX-Fast

You can update an exiting volume to use the PX-Fast functionality by running the following command:

```text
pxctl pxctl volume update --fastpath <px-volume>
```
```output

Volume          	 :  289859383805313381
Name            	 :  v5
Size            	 :  1.0 GiB
Format          	 :  ext4
HA              	 :  1
IO Priority     	 :  LOW
Creation time   	 :  Oct 14 02:35:02 UTC 2022
Shared          	 :  no
Status          	 :  up
State           	 :  detached
Mount Options          	 :  discard
Reads           	 :  0
Reads MS        	 :  0
Bytes Read      	 :  0
Writes          	 :  0
Writes MS       	 :  0
Bytes Written   	 :  0
IOs in progress 	 :  0
Bytes used      	 :  1.0 MiB
Replica sets on nodes:
		Set 0
		  Node 		 : 10.13.161.121 (Pool 06fcc73a-7e2f-4365-95ce-434152789beb )
Replication Status	 :  Detached

Displaying extended volume state:
Fastpath preferred	: true
Fastpath promoted	: false
Fastpath dirty   	: false
Fastpath force failover	: false
Fastpath attached	: FASTPATH_INACTIVE
```

`Fastpath preferred	: true` means that this volume can use PX-Fast functionality if it is a one replica volume and attached locally.