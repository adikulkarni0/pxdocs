---
title: "Storageless nodes with Portworx Enterprise"
hidden: true
keywords: zero storage, storage-less, kubernetes, k8s
description: Run Portworx Enterprise so the storage in a Portworx cluster is consumed by apps running on nodes without storage. Learn how to add a new node with no storage today!
aliases:
    - /portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/storageless-nodes/
---
{{< pxEnterprise >}} can be run in a client-only mode such that the storage available in a Portworx can be consumed by apps that are running on nodes that have no storage. This enables deployments to leverage the powerful {{< pxEnterprise >}} features from any node without having to rely on legacy protocols and adopt containerization faster.

## Add a new storageless node to your Portworx cluster

### Display the status of your Portworx cluster

```text
pxctl status
```

```output
Status: PX is operational
Node ID: a0b87836-f115-4aa2-adbb-c9d0eb597668
	IP: 147.75.104.185
 	Local Storage Pool: 1 pool
	Pool	IO_Priority	Size	Used	Status	Zone	Region
	0	LOW		100 GiB	1.0 GiB	Online	default	default
	Local Storage Devices: 1 device
	Device	Path				Media Type		Size	Last-Scan
	0:1	/dev/mapper/volume-a9e55549	STORAGE_MEDIUM_SSD	100 GiB08 Jan 17 23:15 UTC
	total					-			100 GiB
Cluster Summary
	Cluster ID: bb4bcf13-d394-11e6-afae-0242ac110002
	Node IP: 147.75.104.185 - Capacity: 1.0 GiB/100 GiB Online (This node)
	Node IP: 10.99.117.129 - Capacity: 1.2 GiB/100 GiB Online
	Node IP: 147.75.198.197 - Capacity: 2.0 GiB/320 GiB Online
	Node IP: 10.99.119.1 - Capacity: 1.2 GiB/100 GiB Online
Global Storage Pool
	Total Used    	:  5.3 GiB
	Total Capacity	:  620 GiB

```

### Add a new node to this cluster with no storage

As shown in the command below, a new node is added to the cluster by using the same cluster token which is formed by the cluster id of the existing cluster (which is `bb4bcf13-d394-11e6-afae-0242ac110002` as shown above):

```text
docker run --restart=always --name px-enterprise -d --net=host --privileged=true -v /run/docker/plugins:/run/docker/plugins \
-v /var/lib/osd:/var/lib/osd:shared -v /dev:/dev -v /etc/pwx:/etc/pwx -v /opt/pwx/bin:/export_bin:shared \
-v /var/run/docker.sock:/var/run/docker.sock -v /mnt:/mnt:shared -v /var/cores:/var/cores -v /usr/src:/usr/src \
portworx/px-enterprise -m team0:0 -d team0 -z
```

The `-z` option in the command above starts this node as a zero storage node.

{{<info>}}
**NOTE:** If you already have `config.json` in `/etc/pwx/`, the `config.json` settings will take precedence over the `-z` option. If your deployment or automation scripts place the file on every node, make sure to remove it in order for `-z` option to take effect.
{{</info>}}

### Display the cluster node list

```text
pxctl cluster list
```

```output
Cluster ID: bb4bcf13-d394-11e6-afae-0242ac110002
Status: OK

Nodes in the cluster:
ID					DATA IP		CPU		MEM TOTAL	MEM FREE	CONTAINERS	VERSION		STATUS
a0b87836-f115-4aa2-adbb-c9d0eb597668	147.75.104.185	0.625782	8.4 GB	8.0 GB		N/A		1.1.3-b33d4fa	Online
5de71f19-8ac6-443c-bd83-d3478c485a61	10.99.119.1	0.625		8.4 GB	8.0 GB		N/A		1.1.3-b33d4fa	Online
da758d06-aa9e-4bcb-8cc8-a74ee09030e3	147.75.99.55	16.20603	8.4 GB	8.0 GB		N/A		1.1.3-b33d4fa	Online
a56a4821-6f17-474d-b2c0-3e2b01cd0bc3	147.75.198.197	0.375469	8.4 GB	7.9 GB		N/A		1.1.3-b33d4fa	Online
2c7d4e55-0c2a-4842-8594-dd5084dce208	10.99.117.129	0.749064	8.4 GB	8.0 GB		N/A		1.1.3-b33d4fa	Online

```

### Check node and cluster status

The status below shows that the new node (147.75.99.55) is the zero storage node that has been added to the cluster

```text
pxctl status
```

```output
Status: PX is operational
Node ID: da758d06-aa9e-4bcb-8cc8-a74ee09030e3
	IP: 147.75.99.55
 	Local Storage Pool: 0 pool
	Pool	IO_Priority	Size	Used	Status	Zone	Region
	No storage pool
	Local Storage Devices: 0 device
	Device	Path	Media Type	Size		Last-Scan
	No storage device
	total		-	0 B
Cluster Summary
	Cluster ID: bb4bcf13-d394-11e6-afae-0242ac110002
	Node IP: 147.75.99.55 - Capacity: 0 B/0 B Online (This node)
	Node IP: 147.75.198.197 - Capacity: 2.0 GiB/320 GiB Online
	Node IP: 10.99.117.129 - Capacity: 1.2 GiB/100 GiB Online
	Node IP: 147.75.104.185 - Capacity: 1.0 GiB/100 GiB Online
	Node IP: 10.99.119.1 - Capacity: 1.2 GiB/100 GiB Online
Global Storage Pool
	Total Used    	:  5.3 GiB
	Total Capacity	:  620 GiB

```
