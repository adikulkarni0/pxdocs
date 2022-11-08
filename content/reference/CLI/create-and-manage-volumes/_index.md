---
title: Create and manage volumes using pxctl
keywords: pxctl, command-line tool, cli, reference, create volume, manage volumes, import volume, inspect volume, list volumes, locate volume, restore volume, clone volume, snapshot volume
description: This guide shows you how to create and manage volumes with pxctl.
linkTitle: Create and Manage Volumes
weight: 100
hidesections: true
---

This topic explains how to create and manage volumes using the `pxctl` CLI tool. To view a list of the `pxctl` commands, run one of the following from a worker node:

```text
export PATH=$PATH:/opt/pwx/bin
pxctl volume --help
```

```text
/opt/pwx/bin/pxctl volume --help
```

Alternatively, you can run the following commands anywhere you can access your Kubernetes cluster. First, set an environment variable containing the name of one of your Portworx worker nodes:

```text
PX_POD=$(kubectl get pods -l name=portworx -n kube-system -o jsonpath='{.items[0].metadata.name}')
```

Then you can run `pxctl` commands through `kubectl exec` as follows:

```text
kubectl exec $PX_POD -n kube-system -- /opt/pwx/bin/pxctl volume --help
```

Running the command procudes the following output:

```text
pxctl volume --help
```

```output
Manage volumes

Usage:
  pxctl volume [flags]
  pxctl volume [command]

Aliases:
  volume, v

Examples:
pxctl volume create -s 100 myVolName

Available Commands:
  access               Manage volume access by users or groups
  autofstrim           Periodically frees unused blocks in the volume to the portworx pool
  check                Perform Background filesystem consistency check operation on the volume
  clone                Create a clone volume
  create               Create a volume
  delete               Delete a volume
  ha-update            Update volume HA level
  import               Import data into a volume
  inspect              Inspect a volume
  list                 List volumes in the cluster
  locate               Locate volume
  requests             Show all pending requests
  restore              Restore volume from snapshot
  snap-interval-update Update volume configuration
  snapshot             Manage volume snapshots
  stats                Volume Statistics
  trim                 Frees unused blocks in the volume to the portworx pool
  update               Update volume settings
  usage                Show volume usage information

Flags:
  -h, --help   help for volume

Global Flags:
      --ca string            path to root certificate for ssl usage
      --cert string          path to client certificate for ssl usage
      --color                output with color coding
      --config string        config file (default is $HOME/.pxctl.yaml)
      --context string       context name that overrides the current auth context
  -j, --json                 output in json
      --key string           path to client key for ssl usage
      --output-type string   use "wide" to show more details
      --raw                  raw CLI output for instrumentation
      --ssl                  ssl enabled for portworx

Use "pxctl volume [command] --help" for more information about a command.
```


The following sections explain the individual `pxctl` commands.

## Create volumes

Portworx creates volumes from the global capacity of a cluster. You can expand the capacity and throughput of your cluster by adding new nodes to the cluster. Portworx protects your volumes from hardware and node failures through automatic replication.

Consider the following capabilities of Portworx when creating a new volume using the `pxctl` CLI:

* Durability: Set the replication level through policy, using the `High Availability` setting. For more information, see [storage policy](/reference/cli/storagepolicy).
* Portworx synchronously replicates each write operation to a quorum set of nodes.
* Portworx uses an elastic architecture. This allows you to add capacity and throughput at every layer, at any time.
* Volumes are thinly provisioned. They only use as much storage as needed by the container.
* You can expand and contract the maximum size of a volume, even after you write data to the volume.


To create a volume, run the `pxctl volume create` command and provide the volume name. The following example creates the `myVol` volume:

```text
pxctl volume create myVol
```

```output
3903386035533561360
```

{{<info>}}
Portworx controls the throughput at per-container level and can be shared. Volumes have fine-grained control, set through policy.
{{</info>}}

Before proceeding to the next sections, understand the following concepts:

* Throughput is set by the IO Priority setting. Throughput capacity is pooled.
* If you add a node to the cluster, you expand the available throughput for read and write operations.
* Portworx selects the best node to service read operations, no matter that the operation is from local storage devices or the storage devices attached to another node.
* Read throughput is aggregated, where multiple nodes can service one read request in parallel streams.
* Fine-grained controls: Policies are specified per volume and provide full control to storage.
* Policies enforce how the volume replicates across the cluster, IOPs priority, filesystem, blocksize, and additional parameters described below.
* Policies are specified at create time, and you can apply them to the existing volumes. 

The `pxctl` CLI tool provides options for setting the policies on a volume. Run the `pxctl volume create ` command with the `--help` flag to view the available options:

```text
pxctl volume create --help
```

```output
Create a volume

Usage:
  pxctl volume create [flags]

Aliases:
  create, c

Examples:
pxctl volume create [flags] volume-name

Flags:
  -a, --aggregation_level string            aggregation level (Valid Values: [1 2 3 auto]) (default "1")
      --async_io                            Enable async IO to backing storage
      --best_effort_location_provisioning   requested nodes, zones, racks are optional
  -b, --block_size uint                     block size in Bytes (default 4096)
      --cow_ondemand                        Enable On-demand COW on volume (default true)
  -d, --daily hh:mm,k                       daily snapshot at specified hh:mm,k (keeps 7 by default)
      --direct_io                           Enable Direct IO on volume
      --early_ack                           Reply to async write requests after it is copied to shared memory
      --enforce_cg                          enforce group during provision
      --fastpath                            Enable fastpath IO support for this volume
  -g, --group string                        group
  -h, --help                                help for create
      --io_priority string                  IO Priority (Valid Values: [high medium low]) (default "low")
      --io_profile string                   IO Profile (Valid Values: [sequential cms sync_shared db db_remote auto none]) (default "auto")
      --journal                             Journal data for this volume
  -l, --label pairs                         list of comma-separated name=value pairs
      --max_bandwidth string                maximum io bandwidth (MB/s) this volume is restricted to. syntax --max_bandwidth <ReadBW>,<WriteBW>. provide off for no throttling. e.g. --max_bandwidth 200,off
      --max_iops string                     maximum iops this volume is restricted to. syntax --max_iops <ReadIOPS>,<WriteIOPS>. provide off for no throttling. e.g. --max_iops off,1000
  -m, --monthly day@hh:mm,k                 monthly snapshot at specified day@hh:mm,k (keeps 12 by default)
      --mount_options string                comma separated list of mount options provided as key=value pairs
      --nodes string                        comma-separated Node Ids or Pool Ids
      --nodiscard                           Disable discard support for this volume
  -p, --periodic mins,k                     periodic snapshot interval in mins,k (keeps 5 by default), 0 disables all schedule snapshots
      --policy string                       policy names separated by comma
      --proxy_endpoint string               proxy endpoint provided in the following format '<protocol>://<endpoint>' Ex 'nfs://<nfs-server-ip>'
      --proxy_nfs_exportpath string         export path for nfs proxy volume
      --proxy_nfs_subpath string            sub path from the nfs share to which this proxy volume has access to
  -q, --queue_depth uint                    block device queue depth (Valid Range: [1 256]) (default 128)
      --racks string                        comma-separated Rack names
  -r, --repl uint                           replication factor (Valid Range: [1 3]) (default 1)
      --scale uint                          auto scale to max number (Valid Range: [1 1024]) (default 1)
      --secret_key string                   secret_key to use to fetch secret_data for the PBKDF2 function
      --secret_options string               Secret options is used to pass specific secret parameters. Usage: --secret_options=k1=v1,k2=v2
      --secure                              encrypt this volume using AES-256
      --shared                              make this a globally shared namespace volume
      --sharedv4                            export this volume via Sharedv4
      --sharedv4_failover_strategy string   specifies how aggressively to fail over to a new server for a sharedv4 or sharedv4 service volume (Valid Values: [aggressive normal ])
      --sharedv4_mount_options string       comma separated list of sharedv4 client mount options provided as key=value pairs
      --sharedv4_service_type string        creates a sharedv4 service volume when used with the sharedv4 option (Valid Values: [ClusterIP NodePort LoadBalancer ])
  -s, --size uint                           volume size in GB (default 1)
      --sticky                              sticky volumes cannot be deleted until the flag is disabled
      --storagepolicy string                storage policy name
      --use_cluster_secret                  Use cluster wide secret key to fetch secret_data
  -w, --weekly weekday@hh:mm,k              weekly snapshot at specified weekday@hh:mm,k (keeps 5 by default)
      --zones string                        comma-separated Zone names

Global Flags:
      --ca string            path to root certificate for ssl usage
      --cert string          path to client certificate for ssl usage
      --color                output with color coding
      --config string        config file (default is $HOME/.pxctl.yaml)
      --context string       context name that overrides the current auth context
  -j, --json                 output in json
      --key string           path to client key for ssl usage
      --output-type string   use "wide" to show more details
      --raw                  raw CLI output for instrumentation
      --ssl                  ssl enabled for portworx
```

{{<info>}}
You can also pass these options through the scheduler.
{{</info>}}

### Place a replica on a specific node

Use the `--nodes=LocalNode` flag to create a volume and place at least one replica of the volume on the node where the command is run. This is useful when you use a script to create a volume locally on a node.

{{<info>}}
**NOTE:** You can provide a node ID, node IP address, or pool UUID to the `--nodes` flag.
{{</info>}}

For example, run the following command to create the `localVolume` volume and place a replica of the volume on the local node:

```text
pxctl volume create --nodes=LocalNode localVolume
```

```output
Volume successfully created: 756818650657204847
```

The following command helps you to check if the replica of the volume is on the node where the command was run:

```text
pxctl volume inspect localVolume
```

```output
Volume  :  756818650657204847
        Name                     :  localVolume
        Size                     :  1.0 GiB
        Format                   :  ext4
        HA                       :  1
        IO Priority              :  LOW
        Creation time            :  Mar 20 00:30:05 UTC 2019
        Shared                   :  no
        Status                   :  up
        State                    :  detached
        Reads                    :  0
        Reads MS                 :  0
        Bytes Read               :  0
        Writes                   :  0
        Writes MS                :  0
        Bytes Written            :  0
        IOs in progress          :  0
        Bytes used               :  340 KiB
        Replica sets on nodes:
                Set 0
                  Node           : 70.0.29.90 (Pool 1)
        Replication Status       :  Detached
```

{{<info>}}
The replicas are visible in the `Replica sets on nodes` section of the command output.
{{</info>}}


```text
pxctl volume list --name opt_example
```

```output
ID			NAME		SIZE	HA	SHARED	ENCRYPTED	IO_PRIORITY	STATUS		SNAP-ENABLED
282820401509248281	opt_example	1 GiB	1	no	no		LOW		up - detached	no
```

The `pxctl` CLI utility supports the following key-value pairs:

```
IO priority      - io_priority=[high|medium|low]
Volume size      - size=[1..9][G|M|T]
HA factor        - repl=[1,2,3]
Block size       - bs=[4096...]
Shared volume    - shared=true
File System      - fs=[xfs|ext4]
Encryption       - passphrase=secret
snap_schedule    - "periodic=mins#k;daily=hh:mm#k;weekly=weekday@hh:mm#k;monthly=day@hh:mm#k" where k is the number of snapshots to retain.
```


## The global namespace

{{< content "shared/concepts-shared-volumes.md" >}}


### Related topics

* For information about creating shared Portworx volumes through Kubernetes, refer to the [ReadWriteMany and ReadWriteOnce](/operations/operate-kubernetes/storage-operations/kubernetes-storage-101/volumes/#readwritemany-and-readwriteonce) section.


## Delete volumes

You can delete a specific volume using the `pxctl volume delete ` command. For example, the following command deletes the `myOldVol` volume:

```text
pxctl volume delete myOldVol
```

```output
Delete volume 'myOldVol', proceed ? (Y/N): y
Volume myOldVol successfully deleted.
```

## Import volumes

You can import files from a directory into an existing volume. The existing files on the volume are retained or overwritten.

For example, the following command imports files from the `/path/to/files` directory into `myVol`:

```text
pxctl volume import --src /path/to/files myVol
```

```output
Starting import of  data from /path/to/files into volume myVol...Beginning data transfer from /path/to/files myVol
Imported Bytes :   0% [>---------------------------------------------------------------------------------------------------------------------------------------] 14ms
Imported Files :   0% [>---------------------------------------------------------------------------------------------------------------------------------------] 16ms

Volume imported successfully
```

## Inspect volumes

For more information about inspecting your Portworx volumes, see:

{{< widelink url="/reference/cli/create-and-manage-volumes/inspect-volumes" >}}Inspect volumes{{</widelink>}}

## List volumes

The following command lists all volumes within a cluster:

```text
pxctl volume list
```

```output
ID			NAME		SIZE	HA	SHARED	ENCRYPTED	IO_PRIORITY	STATUS				SNAP-ENABLED
951679824907051932	objectstorevol	10 GiB	1	no	no		LOW		up - attached on 192.168.99.101	no
810987143668394709	testvol		1 GiB	1	no	no		LOW		up - detached			no
1047941676033657203	testvol2	1 GiB	1	no	no		LOW		up - detached			no
800735594334174869	testvol3	1 GiB	1	no	no		LOW		up - detached			no
```

## List volumes in trash can

When the [trash can](/reference/cli/trashcan/) is enabled, you can list volumes in the trash can with the following command:

```text
pxctl volume list --trashcan
```

This displays output similar to the following:

```output
DELETE TIME                   ID                  NAME                  SIZE   HA  SHARED  ENCRYPTED  IO_PRIORITY  STATUS         DELETE_TIMER
Tue Mar 29 21:51:16 UTC 2022  780196670250220779  newvol-tc-1648590676  1 GiB  1   no      no         LOW          up - detached  9m33s
```

In the sample output above, the trash can volume name is in the format `<original-vol-name>-tc-<original-vol-id>`.

### Related topics

* To learn how to enable the volume trash can feature on a cluster, see [Volume trash can](/reference/cli/trashcan/).

## Locate volumes

The `pxctl volume locate` command displays the mounted location of a volume in the containers running on the node:

```text
pxctl volume locate 794896567744466024
```

```output
host mounted:
  /directory1
  /directory2
```

In the above example, the `794896567744466024` volume is mounted in two containers through the `/directory1` and `/directory2` mount points.

## Create volume snapshots

{{< content "shared/reference-CLI-intro-snapshots.md" >}}

{{< content "shared/reference-CLI-creating-snapshots.md" >}}

Snapshots are read-only. To restore a volume from a snapshot, use the `pxctl volume restore` command.


### Related topics

* For information about creating snapshots of your Portworx volumes through Kubernetes, refer to the [Create and use snapshots](/operations/operate-kubernetes/storage-operations/create-snapshots/) page.

## Clone volumes

Use the `pxctl volume clone` command to create a volume clone from a volume or snapshot.

To view the in-built help, run the `pxctl volume clone` command with the `--help` flag:


```text
pxctl volume clone --help
```

```output
Create a clone volume

Usage:
  pxctl volume clone [flags]

Aliases:
  clone, cl

Examples:
pxctl volume clone [flags] volName

Flags:
  -h, --help           help for clone
  -l, --label string   list of comma-separated name=value pairs
      --name string    clone name

Global Flags:
      --ca string            path to root certificate for ssl usage
      --cert string          path to client certificate for ssl usage
      --color                output with color coding
      --config string        config file (default is $HOME/.pxctl.yaml)
      --context string       context name that overrides the current auth context
  -j, --json                 output in json
      --key string           path to client key for ssl usage
      --output-type string   use "wide" to show more details
      --raw                  raw CLI output for instrumentation
      --ssl                  ssl enabled for portworx
```


For example, the following command creates the `myvol_clone` clone from the parent `myvol` volume:


```text
pxctl volume clone -name myvol_clone myvol
```

```output
Volume clone successful: 55898055774694370
```

### Related topics

* For information about creating a clone from a snapshot through Kubernetes, refer to the [On-demand snapshots](/operations/operate-kubernetes/storage-operations/create-snapshots/on-demand/) page.

## Restore a volume

{{< content "shared/reference-CLI-restore-volume-from-snapshot.md" >}}


### Related topics

* To learn how to enable the volume trash can feature on a cluster, see [Volume trash can](/reference/cli/trashcan/).
* For information about restoring a Portworx volume with data from a snapshot through Kubernetes, refer to the [Restore snapshots](/operations/operate-kubernetes/storage-operations/kubernetes-storage-101/snapshots/#restore-snapshots) page.


## Update the snap interval of a volume

Please see the documentation for [snapshots] (/reference/cli/snapshots) for more details.

### Related topics

* For information about creating scheduled snapshots of a Portworx volume through Kubernetes, refer to the [Scheduled snapshots](/operations/operate-kubernetes/storage-operations/create-snapshots/scheduled/) page.

## Show volume stats

The `pxctl volume stat` command displays the real-time read/write IO throughput:

```text
pxctl volume stats mvVol
```

```output
TS			Bytes Read	Num Reads	Bytes Written	Num Writes	IOPS		IODepth		Read Tput	Write Tput	Read Lat(usec)	Write Lat(usec)
2019-3-4:11 Hrs		0 B		0		0 B		0		0		0		0 B/s		0 B/s		0		0
```

## Manage volume access rules

Using `pxctl`, you can manage your volume access rules. See the [volume access](/reference/cli/volume-access) page for more details.

## Update the replication factor of a volume

You can use the `pxctl volume ha-update` to increase or decrease the replication factor of a Portworx volume. Refer to the [update volumes](/reference/cli/updating-volumes#update-a-volume-s-replication-factor) page for more details.

## Update the settings of a volume

Using the `pxctl volume update` command, you can update the settings of your Portworx volumes. Refer to the [updating volumes](/reference/cli/updating-volumes) page for additional details.

## Volume usage

To get more information about the usage of your Portworx volumes, run the `pxctl volume usage` command with the name or the ID of your volume:

```text
pxctl volume usage 13417687767517527
```

## Understand copy-on-write features

By default, Portworx uses features present in the underlying file system to take snapshots through copy-on-write and checksum storage blocks.

When using the copy-on-write feature to take snapshots, overwriting a block does not update it in place. Instead, every overwrite allocates or updates a new block, and the filesystem metadata is updated to point to this new block. This technique is called redirect-on-write. When using this feature, a block overwrite almost always involves block updates in multiple areas: 

* the target block
* any linked indirect file blocks
* filesystem metadata blocks
* filesystem metadata indirect file blocks

In a background process, separate from the overwrite operation, the old block is freed only if it is not referred by a snapshot or clone. In addition to copy-on-write, the file system checksums all blocks to detect lost writes and stores the checksum values in a different location away from their associated data.

While these combined features increase the integrity of the data stored on the filesystem, they also increase the read and write overhead on the drives that use them. This slows down performance and increases latency during file operations.

Depending on your use-case, you can trade off the integrity copy-on-write features offer for increased performance and lower latency. You can do this on a per-volume basis using the `pxctl` command.

### Disable copy-on-write features for a volume

To disable copy-on-write features for a specific volume, use the `pxctl volume update` command with the `--cow_ondemand off` flag.

  ```text
  pxctl volume update  --cow_ondemand off <volume-ID>
  ```
  ```output
  Update Volume: Volume update successful for volume 850767800314736346
  ```

