---
title: Scale down nodes
keywords: scale down, scaling
description: Learn how to remove an offline node from a cluster.
aliases:
    - /install-with-other/operate-and-maintain/scaling/scale-down/
---
<!--

TODO: I'm commenting this out for now. We should probably revisit it as part of the reorg.

{{<info>}}
This document presents the **non-Kubernetes** method of removing a node. Please refer to the [Decommission a Node](/operations/operate-kubernetes/uninstall/decommission-a-node/) page if you are running Portworx on Kubernetes.
{{</info>}}
-->

## Removing offline nodes

This document describes how to remove an offline node from a cluster. If you are specifically interested in decommissioning nodes with no storage that have been offline for an extended period, see [Automatic decommission of storage less nodes](#automatic-decommission-of-storage-less-nodes).

1. Identify the cluster that needs to be managed:

    ```text
    pxctl status
    ```

    ```output
    Status: PX is operational
    Node ID: a56a4821-6f17-474d-b2c0-3e2b01cd0bc3
        IP: 147.75.198.197
        Local Storage Pool: 2 pools
        Pool	IO_Priority	Size	Used	Status	Zone	Region
        0	LOW		200 GiB	1.0 GiB	Online	default	default
        1	LOW		120 GiB	1.0 GiB	Online	default	default
        Local Storage Devices: 2 devices
        Device	Path				Media Type		SizLast-Scan
        0:1	/dev/mapper/volume-27dbb728	STORAGE_MEDIUM_SSD	200 GiB		08 Jan 17 05:39 UTC
        1:1	/dev/mapper/volume-0a31ef46	STORAGE_MEDIUM_SSD	120 GiB		08 Jan 17 05:39 UTC
        total					-			320 GiB
    Cluster Summary
        Cluster ID: bb4bcf13-d394-11e6-afae-0242ac110002
        Node IP: 147.75.198.197 - Capacity: 2.0 GiB/320 GiB Online (This node)
        Node IP: 10.99.117.129 - Capacity: 1.2 GiB/100 GiB Online
        Node IP: 10.99.119.1 - Capacity: 1.2 GiB/100 GiB Online
    Global Storage Pool
        Total Used    	:  4.3 GiB
        Total Capacity	:  520 GiB
    ```

2. List the nodes in the cluster:

    ```text
    pxctl cluster list
    ```

    ```output
    Cluster ID: bb4bcf13-d394-11e6-afae-0242ac110002
    Status: OK

    Nodes in the cluster:
    ID					DATA IP		CPU		MEM TOTAL	MEM FREE	CONTAINERS	VERSION		STATUS
    a56a4821-6f17-474d-b2c0-3e2b01cd0bc3	147.75.198.197	1.629073	8.4 GB		7.9 GB		N/A		1.1.2-c27cf42	Online
    2c7d4e55-0c2a-4842-8594-dd5084dce208	10.99.117.129	0.125156	8.4 GB		8.0 GB		N/A		1.1.3-b33d4fa	Online
    5de71f19-8ac6-443c-bd83-d3478c485a61	10.99.119.1	0.25		8.4 GB		8.0 GB		N/A		1.1.3-b33d4fa	Online
    ```

3. List the volumes in the cluster:

    ```text
    pxctl volume list
    ```

    ```output
    ID			NAME	SIZE	HA	SHARED	ENCRYPTED	PRIORITSTATUS
    845707146523643463	testvol	1 GiB	1	no	no		LOW	up - attached on 147.75.198.197
    ```

    In this case, there is one volume in the cluster and it is attached to node with IP 147.75.198.97

4. Identify the node that you want to remove from the cluster.<br><br>

    In the following example, node 147.75.198.197 has been marked as offline:

    ```text
    pxctl cluster list
    ```

    ```output
    Cluster ID: bb4bcf13-d394-11e6-afae-0242ac110002
    Status: OK

    Nodes in the cluster:
    ID					DATA IP		CPU		MEM TOTAL	MEM FREE	CONTAINERS	VERSION		STATUS
    2c7d4e55-0c2a-4842-8594-dd5084dce208	10.99.117.129	5.506884	8.4 GB	8.0 GB		N/A		1.1.3-b33d4fa	Online
    5de71f19-8ac6-443c-bd83-d3478c485a61	10.99.119.1	0.25		8.4 GB	8.0 GB		N/A		1.1.3-b33d4fa	Online
    a56a4821-6f17-474d-b2c0-3e2b01cd0bc3	147.75.198.197	-		-	N/A		1.1.2-c27cf42	Offline
    ```

5. Attach and detach the volume in one of the surviving nodes:

    ```text
    pxctl host attach 845707146523643463
    ```

    ```output
    Volume successfully attached at: /dev/pxd/pxd845707146523643463
    ```

    ```text
    pxctl host detach 845707146523643463
    ```

    ```output
    Volume successfully detached
    ```

6. Delete the local volume that belonged to the offline node:

    ```text
    pxctl volume delete 845707146523643463
    ```

    ```output
    Volume 845707146523643463 successfully deleted.
    ```

7. Delete the node that is offline:

    ```text
    pxctl cluster delete a56a4821-6f17-474d-b2c0-3e2b01cd0bc3
    ```

    ```output
    Node a56a4821-6f17-474d-b2c0-3e2b01cd0bc3 successfully deleted.
    ```

8. List the nodes in the cluster to make sure that the node is removed:

    ```text
    pxctl cluster list
    ```

    ```output
    Cluster ID: bb4bcf13-d394-11e6-afae-0242ac110002
    Status: OK

    Nodes in the cluster:
    ID					DATA IP		CPU		MEM TOTAL	MEM FREE	CONTAINERS	VERSION		STATUS
    2c7d4e55-0c2a-4842-8594-dd5084dce208	10.99.117.129	4.511278	8.4 GB	8.0 GB		N/A		1.1.3-b33d4fa	Online
    5de71f19-8ac6-443c-bd83-d3478c485a61	10.99.119.1	0.500626	8.4 GB	8.0 GB		N/A		1.1.3-b33d4fa	Online
    ```

9. Show the cluster status:

    ```text
    pxctl status
    ```

    ```output
    Status: PX is operational
    Node ID: 2c7d4e55-0c2a-4842-8594-dd5084dce208
        IP: 147.75.198.199
        Local Storage Pool: 1 pool
        Pool	IO_Priority	Size	Used	Status	Zone	Region
        0	LOW		100 GiB	1.2 GiB	Online	default	default
        Local Storage Devices: 1 device
        Device	Path				Media Type		Size	Last-Scan
        0:1	/dev/mapper/volume-9f6be49c	STORAGE_MEDIUM_SSD	100 GiB08 Jan 17 06:34 UTC
        total					-			100 GiB
    Cluster Summary
        Cluster ID: bb4bcf13-d394-11e6-afae-0242ac110002
        Node IP: 10.99.117.129 - Capacity: 1.2 GiB/100 GiB Online (This node)
        Node IP: 10.99.119.1 - Capacity: 1.2 GiB/100 GiB Online
    Global Storage Pool
        Total Used    	:  2.3 GiB
        Total Capacity	:  200 GiB
    ```

## Removing a functional node from a cluster

A functional Portworx node may need to be removed from the cluster. In this section, we'll demonstrate the removal of a node by running commands on the node itself as well as the removal of a node from another node.

The following output shows the state of the cluster and the different node IPs and node IDs:

```text
pxctl status
```

```output
Status: PX is operational
Node ID: 5f8b8417-af2b-4ea7-930e-0027f6bbcbd1
        IP: 172.31.46.119
        Local Storage Pool: 1 pool
        POOL    IO_PRIORITY     SIZE    USED    STATUS  ZONE    REGION
        0       LOW             64 GiB  11 GiB  Online  c       us-east-1
        Local Storage Devices: 1 device
        Device  Path            Media Type              Size            Last-Scan
        0:1     /dev/xvdf       STORAGE_MEDIUM_SSD      64 GiB          25 Feb 17 21:13 UTC
        total                   -                       64 GiB
Cluster Summary
        Cluster ID: 0799207a-eec6-4fc6-a5f1-d4a612b74cc3
        IP              ID                                      Used    Capacity        Status
        172.31.40.38    ec3ed4b9-68d5-4e83-a7ce-2bc112f5f131    11 GiB  64 GiB          Online
        172.31.37.211   17a6fb2c-0d19-4bae-a73f-a85e0514ae8b    11 GiB  64 GiB          Online
        172.31.35.130   a91175b6-ff69-4eff-8b7f-893373631483    11 GiB  64 GiB          Online
        172.31.45.106   048cc2f8-022e-47d9-b600-2eeddcd64d51    11 GiB  64 GiB          Online
        172.31.45.56    f9cb673e-adfa-4e4f-a99a-ec8e1420e645    11 GiB  64 GiB          Online
        172.31.46.119   5f8b8417-af2b-4ea7-930e-0027f6bbcbd1    11 GiB  64 GiB          Online (This node)
        172.31.39.201   355ee6aa-c7eb-4ac6-b16b-936b1b58aa24    11 GiB  64 GiB          Online
        172.31.33.151   871c503d-fa6e-4599-a533-41e70a72eafd    11 GiB  64 GiB          Online
        172.31.33.252   651ca0f4-c156-4a14-b2f3-428e727eb6b8    11 GiB  64 GiB          Online
Global Storage Pool
        Total Used      :  99 GiB
        Total Capacity  :  576 GiB
```

### Suspend active cloudsnap operations

1. Identify any active cloudsnap operations being run on the node that you intend to decommission:

    ```text
    pxctl cloudsnap status
    ```

    The `STATE` of active operations shows as `Backup-Active`:
    
    ```output
    NAME                                    SOURCEVOLUME        STATE           NODE            TIME-ELAPSED        COMPLETED   
    0ddf4424-c7a7-4435-a80d-278535e49860    885345022234521857  Backup-Done     10.13.90.125    39.746191264s       Tue, 22 Mar 2022 22:53:37 UTC
    31ab4e56-f0d2-4bdf-a236-3c3ccf47f276    186701534582547510  Backup-Done     10.13.90.122    1.677455484s        Tue, 22 Mar 2022 23:59:49 UTC
    274c0d4c-12a4-403f-8b0b-73176c2d03e2    885345022234521857  Backup-Done     10.13.90.125    27.550329395s       Wed, 23 Mar 2022 00:00:15 UTC
    ee91a94e-b311-4c4d-ba02-2307865c1b93    649554470078043771  Backup-Active   10.13.90.125    5m12.61653365s

    ```

    From this output, identify the volumes with active backups. For example, if node 10.13.90.125 is being decommissioned, then the volume with active backup is `649554470078043771`.

1. Identify the namespace of the volume the cloudsnap operation is occuring on. The namespace is displayed under the `Labels` section in the output from the following command. Replace `<source_volume>` with the `SOURCEVOLUME` value for the volume that is in a `Backup-Active` state from the previous output:

    ```text
    pxctl volume inspect <source_volume>
    ```
    ```output
    Volume               :  649554470078043771
    Name                 :  pvc-d509932e-7772-4ac5-9bc0-d4827680f6de
    Size                 :  500 GiB
    Format               :  ext4
    HA                   :  3
    IO Priority          :  LOW
    Creation time        :  Mar 22 20:37:52 UTC 2022
    Shared               :  v4 (service)
    Status               :  up
    State                :  Attached: 78dbf17e-28b1-4c78-b35b-10f4e076cac8 (10.13.90.119)
    Last Attached        :  Mar 22 20:37:58 UTC 2022
    Device Path          :  /dev/pxd/pxd649554470078043771
    Labels               :  mount_options=nodiscard=true,namespace=vdbench,nodiscard=true,pvc=vdbench-pvc-sharedv4,repl=3,sharedv4=true,sharedv4_svc_type=ClusterIP
    ...
    ```

1. Suspend backup operations for the volume and wait for current backup to complete:

    ```text
    storkctl suspend volumesnapshotschedule vdbench-pvc-sharedv4-schedule  -n vdbench
    ```

1. Verify the suspension. The `SUSPEND` field will show as `true`:

    ```text
    storkctl get volumesnapshotschedule -n vdbench 
    ```
    ```output
    NAME                            PVC                    POLICYNAME   PRE-EXEC-RULE   POST-EXEC-RULE   RECLAIM-POLICY   SUSPEND   LAST-SUCCESS-TIME
    vdbench-pvc-sharedv4-schedule   vdbench-pvc-sharedv4   testpolicy                                    Delete           true      22 Mar 22 17:10 PDT
    ```

Repeat these steps until all active snaps complete and all backup operations are suspended on the node that you want to decommission.

### Prevention of data loss

If any node hosts a volume with replication factor of 1, then Portworx disallows decommissioning of such nodes because there is data loss.

One possible workaround to decommission such a node is to increase the replication of single replica volumes by running `volume ha-update`.

1. List all the volumes hosts on the decommisioning node:

    ```text
    pxctl volumev  list --node 048cc2f8-022e-47d9-b600-2eeddcd64d51
    ```
    ```output
    ID                      NAME                                            SIZE    HA      SHARED  ENCRYPTED  PROXY-VOLUME     IO_PRIORITY     STATUS                          SNAP-ENABLED
    633738568577538909      pvc-d509932e-7772-4ac5-9bc0-d4827680f6de        2 GiB   3       no      no         no               LOW             up - attached on 172.31.45.106    no
    161898313715947409      pvc-2d114856-2df4-47e9-b2af-68f0d970e10c        2 GiB   1       no      no         no               LOW             up - attached on 172.31.45.106    no
    ```

2. Increase the replication factor:

    ```text
    pxctl volume ha-update --repl 2 161898313715947409
    ```

Once the volume is completely replicated onto another node, continue with the node decommissioning. This time, the volume already has another replica on another node, so decommissioning the node will reduce the replication factor of the volume and remove the node.

### Placing the node in maintenance mode

<!-- When does the user want to do this?  Why are we telling them about it in a separate section instead of inline in the numbered instructions? -->

After identifying the node to be removed (see "Identify the node that you want to remove from the cluster" above), place the node in maintenance mode.

Log in to the node to be decommissioned:

```text
pxctl service maintenance --enter
```

```output
This is a disruptive operation, PX will restart in maintenance mode.
Are you sure you want to proceed ? (Y/N): y
Entered maintenance mode.
```

### Run the cluster delete command

<!-- These examples use the same commands. Why are there two examples? -->

**Example 1: Running the cluster delete command from a different node**

`ssh` to `172.31.46.119` and run the following command:
<!-- is it always this specific IP, or does this change based on user configurations? -->

```text
pxctl cluster delete 048cc2f8-022e-47d9-b600-2eeddcd64d51
```

```output
Node 048cc2f8-022e-47d9-b600-2eeddcd64d51 successfully deleted.
```

**Example 2: Running the cluster delete command from the same node**

`ssh` to `172.31.33.252` and type:

```text
pxctl cluster delete 651ca0f4-c156-4a14-b2f3-428e727eb6b8
```

```output
Node 651ca0f4-c156-4a14-b2f3-428e727eb6b8 successfully deleted.
```

### Clean up Portworx metadata on the node

To learn how to remove or clean up Portworx metadata on the decommmisioned node, see [clean up Portworx metadata on the node](/operations/operate-kubernetes/k8s-node-rejoin/#clean-up-portworx-metadata-on-the-node).

## Automatic decommission of storage less nodes

 Storage less nodes that are initialized and added to the cluster may not be needed once they complete their tasks (such as in a scheduler workflow). If they are taken offline or destroyed, the cluster will still retain the nodes and mark them as offline.
 
 If eventually a majority of such nodes exist, the cluster won't have a quorum of nodes that are online. The solution is to run cluster delete commands and remove such nodes. This gets more laborious with more such nodes or an increased frequency of such nodes added and taken down.
 
 To help with this, Portworx waits until a grace period of 48 hours has passed. After this period, offline nodes with no storage will be removed from the cluster. There is no CLI command needed to turn on or trigger this feature.