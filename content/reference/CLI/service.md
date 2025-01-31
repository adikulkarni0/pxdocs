---
title: Service operations using pxctl
linkTitle: Service
keywords: pxctl, command-line tool, cli, reference, service, troubleshooting, diagnostics, call-home, maintenance mode
description: How to use the pxctl service command.
weight: 1600
---

The Portworx `pxctl` CLI tool allows you to run the following service operations:

- Perform a node audit
- Manage the call home feature
- Generate diagnostics package
- Get the version of the installed software
- Configure kvdb
- Place Portworx in maintenance mode
- Manage the physical storage drives

These commands are helpful when you want do debug issues related to your _Portworx_ cluster.

You can see an overview of the available service operations by running:

```text
/opt/pwx/bin/pxctl service --help
```

```output
Service mode utilities

Usage:
  pxctl service [flags]
  pxctl service [command]

Aliases:
  service, sv

Available Commands:
  audit          Audit the PX node
  call-home      Enable or disable the call home feature
  diags          creates a new tgz package with essential diagnostic information.
  drive          Storage drive maintenance
  email          Email setting commands
  exit           Stop the PX daemon
  info           Show PX module version information
  kvdb           PX Kvdb operations
  maintenance    Maintenance mode operations
  node           Node maintenance operations
  node-usage     Volume usage by PX node
  node-wipe      Wipes PX configuration data on this node
  pool           Storage pool maintenance
  relaxedreclaim Manage the RelaxedReclaim queue for the node
  slack          Slack setting commands
  usage-report   creates & download tgz package for px consumption reports

Flags:
  -h, --help   help for service

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

Use "pxctl service [command] --help" for more information about a command.
```

## Perform a node audit

You can audit the node with:

```text
pxctl service audit
```

```output
AuditID		Error	Message
kvdb-limits	none	KV limits audit not yet available

kvdb-response	none	KV response audit not yet available
```

## Manage the call home feature

With `pxctl`, you can enable and disable the call home feature:

```text
pxctl service call-home enable
```

```output
Call home feature successfully enabled
```

If you want to disable this feature, just run:

```text
pxctl service call-home disable
```

```output
Call home feature successfully disabled
```

## Generate a complete diagnostics package

When there is an operational failure, you can use the `pxctl service diags` command to generate a complete diagnostics package. 

The diagnostics package will be available at /var/cores. It will be automatically uploaded to Pure1 if telemetry is enabled. 

See [Auto-uploading diags using Pure1 integration ](/operations/operate-kubernetes/troubleshooting/collecting-diagnostics/#auto-uploading-diags-using-pure1-integration) for details on enabling Pure1 telemetry.

```text
pxctl service diags --help
```

```output
creates a new tgz package with essential diagnostic information.

Usage:
  pxctl service diags [flags]

Aliases:
  diags, d

Flags:
  -a, --all                 creates a new tgz package with all the available diagnostic information.
  -c, --cluster             generate diags for all the nodes in the cluster.
      --container string    PX container ID
      --dockerhost string   docker host daemon (default "unix:///var/run/docker.sock")
  -f, --force               force overwrite existing diags.
  -h, --help                help for diags
  -l, --live                gets diags from running px
  -n, --node string         generate diags for a specific remote node with the provided NodeIp or NodeID.
  -o, --output string       output file name
  -p, --profile             only dump profile
  -u, --upload              upload diags to cloud

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

As an example, here's how to generate the diagnostics package for a container called `px-enterprise`:

```text
pxctl service diags -a
```
```output
Connected to Docker daemon.  unix:///var/run/docker.sock
Running PX diagnostics on local node....
Using PX OCI container: 2.8.0.0-c60727b
Archived cores to: /var/cores/test-k8s1-node0-px-cores.20210514230349.tgz, cleaning up archived cores...
Removing /var/cores/core-px-storage-sig6-user0-group0-pid312-time1620773250...
Getting diags file....
Creating a diags tar ball...
Executing core cleanup...
Finished core cleanup.
Removing /var/cores/test-k8s1-node0-px-cores.20210514230349.tgz...
Generated diags:  /var/cores/test-k8s1-node0-diags-20210514230401.tar.gz
Done generating PX diagnostics.
```

## Get the version of the installed software

The following command displays the version of the installed software:

```text
pxctl service info
```

```output
PX (OCI) Version:  2.0.2.1-1d83ac2
PX (OCI) Build Version:  1d83ac2baeb27451222edcd543249dd2c2f941e4
PX Kernel Module Version:  72D3C244593F45167A6B49D
```

## Configure KVDB

You can configure the KVDB with the `pxctl service kvdb` command. To get an overview of the available subcommands, run:

```text
pxctl service kvdb --help
```

```output
PX Kvdb operations

Usage:
  pxctl service kvdb [flags]
  pxctl service kvdb [command]

Aliases:
  kvdb, k

Available Commands:
  endpoints      List the kvdb client endpoints
  members        List the kvdb cluster members
  restore        Restore keys and values into kvdb from a kvdb.dump file

Flags:
  -h, --help   help for kvdb

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

Use "pxctl service kvdb [command] --help" for more information about a command.
```

## Place Portworx in maintenance mode

Use the `service maintenance` command to enter or exit maintenance mode. Once the node is in maintenance mode, you can add or replace drives, add memory, and so on.

To list the available subcommands, run the following:

```text
pxctl service maintenance --help
```

```output
Maintenance mode operations

Usage:
  pxctl service maintenance [flags]

Aliases:
  maintenance, m

Flags:
  -c, --cycle   cycle maintenance mode
  -e, --enter   enter maintenance mode
  -x, --exit    exit maintenance mode
  -h, --help    help for maintenance

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

Enter maintenance mode with:

```text
pxctl service maintenance --enter
```

```output
This is a disruptive operation, PX will restart in maintenance mode.
Are you sure you want to proceed ? (Y/N): y
```

Once you're done adding or replacing drives, or adding memory, you can exit maintenance mode by running:

```text
pxctl service maintenance --exit
```

## Manage the physical storage drives

You can manage the physical storage drives on a node using the `pxctl service drive` command:

```text
pxctl service drive --help
```

```output
Storage drive maintenance

Usage:
  pxctl service drive [flags]
  pxctl service drive [command]

Available Commands:
  add         Add storage
  check       Check drives
  rebalance   Rebalance storage
  replace     Replace source drive with target drive
  show        Show drives

Flags:
  -h, --help   help for drive

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

Use "pxctl service drive [command] --help" for more information about a command.
```

### Add a physical drive to a server

Use the `pxctl sv drive add` command to add a physical drive to a server. To see an overview of the available flags, run:

```text
pxctl service drive add --help
```

```output
Add storage

Usage:
  pxctl service drive add [flags]

Flags:
      --cache int          Use this drive as a cache device for given pool. (default -1)
  -d, --drive string       comma-separated source drives
  -h, --help               help for add
      --journal            Use this drive as a journal device.
      --metadata           Use this drive as a system metadata device.
  -o, --operation string   start|status (Valid Values: [start status]) (default "start")
  -s, --spec string        Cloud drive spec in type=<>,size=<> format

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

You can add physical drives to a server using the `pxctl service drive add` command. The following example shows how to add a physical drive:

```text
pxctl service drive add --drive /dev/mapper/volume-3bfa72dd -o start
```

```output
Adding drives may make storage offline for the duration of the operation.
Are you sure you want to proceed ? (Y/N): y
Adding device  /dev/mapper/volume-3bfa72dd ...
Drive add  successful. Requires restart.
```

{{<info>}}
**NOTE:** To add physical drives, you must place the server in [maintenance mode](#place-portworx-in-maintenance-mode) first.
{{</info>}}

## Rebalance storage across drives

Over time, your cluster's storage may become unbalanced, with some pools and drives filled to capacity and others utilized less. This can occur for a number of reasons, such as:

* Adding new nodes or pools
* Increasing the size of pools by increasing the size of underlying drives or adding new drives
* Volumes being removed from only a subset of nodes

If your cluster's storage is unbalanced, you can use the `pxctl service pool rebalance` command to redistribute the volume replicas. This command determines which pools are over-loaded and under-loaded, and moves volume replicas from the former to the latter. This ensures that all pools on the nodes are evenly loaded.

{{<info>}}
**NOTE:** You can run this cluster-wide command from any of your Portworx nodes.
{{</info>}}

### Start a rebalance operation

Use the `submit` subcommand to start the rebalance operation, which returns a job ID:

```text
pxctl service pool rebalance submit
```
```output
This command will start rebalance for:
 - all storage pools for checking if they are over-loaded
 - all storage pools for checking if they are under-loaded
which meet either of following conditions:

1. Pool's provision space is over 20% or under 20% of mean value across pools
2. Pool's used space is over 20% or under 20% of mean value across pools
*Note: --remove-repl-1-snapshots is off, space from such snapshots will not be reclaimed

Do you wish to proceed ?  (Y/N): Y
Pool rebalance request: 859941020356581382 submitted successfully.
For latest status: pxctl service pool rebalance status --job-id 859941020356581382
```

{{<info>}}
**NOTE:** The rebalance operation runs as a background service.
{{</info>}}

### List running rebalance jobs

Enter the `list` subcommand to see all currently running jobs:

```text
pxctl service pool rebalance list
```
```output
JOB			STATE	CREATE TIME			STATUS
859941020356581382	RUNNING	2020-08-11T11:16:12.928785518Z
```

### Monitor a rebalance operation

Monitor the status of a rebalance operation, as well as all the steps it has taken so far, by entering the `status` subcommand with the `--job-id` flag and the ID of a running rebalance job:

```text
pxctl service pool rebalance status --job-id 859941020356581382
```
```output
Rebalance summary:
Job ID             : 859941020356581382
Job State          : DONE
Last updated       : Sun, 23 Aug 2020 22:08:31 UTC
Total running time : 4 minutes and 25 seconds
Job summary
    - Provisioned space balanced : 827 GiB done, 0 B pending
    - Used space balanced        : 17 GiB done, 0 B pending
    - Volume replicas balanced   : 42 done, 0 pending
Rebalance actions:
Replica add action:
        Volume             : 956462089713112944
        Pool               : 728cb696-c6e3-417e-a269-eaca75365214
        Node               : d186025f-a89c-4e29-bb7e-489393d6636b
        Replication set ID : 0
        Start              : Sun, 23 Aug 2020 22:04:06 UTC
        End                : Sun, 23 Aug 2020 22:04:27 UTC
        Work summary
            - Provisioned space balanced : 20 GiB done, 0 B pending
Replica remove action:
        Volume             : 956462089713112944
        Pool               : 51b79960-39d4-4c71-8c5d-5e34a2b712e3
        Node               : 7a1d3cfb-1343-402c-b0e1-81b36fef6177
        Replication set ID : 0
        Start              : Sun, 23 Aug 2020 22:04:06 UTC
        End                : Sun, 23 Aug 2020 22:04:29 UTC
        Work summary
            - Provisioned space balanced : 20 GiB done, 0 B pending
```

### Pause or terminate a running rebalance operation

If you need to temporarily suspend a running rebalance operation, you can pause it. Otherwise, you can cancel it entirely:

* Use the `cancel` subcommand subcommand with the `--job-id` flag and the ID of a running rebalance job to terminate a running rebalance operation:

    ```text
    pxctl service pool rebalance cancel --job-id 859941020356581382
    ```

* Use `pause` subcommand subcommand with the `--job-id` flag and the ID of a running rebalance job to terminate a running rebalance operation:

    ```text
    pxctl service pool rebalance pause --job-id 859941020356581382
    ```

### pxctl service pool rebalance reference

Rebalance storage pools

```
pxctl service pool rebalance [command] [flags]
```

#### Commands

|**Command**|**Description**|
|----|----|
| cancel |    Cancels a rebalance job specified with the `--job-ID` flag and a valid job ID |
| list   |    Lists rebalance jobs in the system |
| pause  |    Pauses a rebalance job specified with the `--job-ID` flag and a valid job ID |
| resume |    Resumes a rebalance job specified with the `--job-ID` flag and a valid job ID  |
| status |    Shows the status of a rebalance job specified with the `--job-ID` flag and a valid job ID |
| submit |    Start a new rebalance job |

## Display drive information

You can use the `pxctl service drive show` command to display drive information on the server:

```text
pxctl service drive show
```

```output
PX drive configuration:
Pool ID: 0 
	Type:  Default 
	UUID:  d5b8cfd8-3b48-4d06-ab73-2b69eeebb81b 
	IO Priority:  HIGH 
	Labels:  medium=STORAGE_MEDIUM_MAGNETIC,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,iopriority=HIGH,kubernetes.io/arch=amd64,kubernetes.io/hostname=myhostname-k8s1-node0,kubernetes.io/os=linux 
	Size: 3.0 TiB 
	Status: Online 
	Has metadata:  Yes 
	Balanced:  Yes 
	Drives:
	3: /dev/vdd, Total size 1.0 TiB, Online
	1: /dev/vdb, Total size 1.0 TiB, Online
	2: /dev/vdc, Total size 1.0 TiB, Online
	Cache Drives:
	No Cache drives found in this pool
```
<!-- need example output that includes caching -->

## Configure the email settings for alerts

You can use the `pxctl service email` command to list the available subcommands:

```text
pxctl service email
```

```output
Usage:
  pxctl service email [flags]
  pxctl service email [command]

Available Commands:
  clear       Clear email settings for alerts.
  get         Get email settings for alerts.
  set         Configure email settings for alerts.

Flags:
  -h, --help   help for email

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

#### pxctl service email set

```text
pxctl service email set --help
```
```output
Configure email settings for alerts.

Usage:
  pxctl service email set [flags]

Aliases:
  set, s

Flags:
  -h, --help               help for set
      --password string    Password to authenticate with smtp-server (if required)
  -r, --recipient string   Recipient of alert emails.
      --sender string      Sender of alert emails (defaults to noreply@portworx.com) (default "noreply@portworx.com")
  -s, --server string      IP or DNS name for smtp server
      --severity string    Minimum severity for email trigger, (warning|critical) (default "critical")
  -p, --smtp-port string   IP or DNS name for smtp server
  -u, --username string    Username to authenticate with smtp-server (if required)

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

Receive Warning and Critical alerts:

```text
/opt/pwx/bin/pxctl service email set --server=smtp.gmail.com --smtp-port=587 --username=me@portworx.com --password='IncrediblySecretPassword' --recipient=me@portworx.com --severity warning
```

Receive Only Critical alerts:

```text
/opt/pwx/bin/pxctl service email set --server=smtp.gmail.com --smtp-port=587 --username=me@portworx.com --password='IncrediblySecretPassword' --recipient=me@portworx.com
```

{{<info>}}
**NOTE:** You must add single quotes around the password.
{{</info>}}

<!--

Opt-out of Portworx Support receiving critical alerts:

`/opt/pwx/bin/pxctl service email set --server=smtp.gmail.com --smtp-port=587 --username=me@portworx.com --password='IncrediblySecretPassword' --recipient=me@portworx.com --notify=false`

--notify false is a hidden command

-->

## Scan for bad blocks

You can use `pxctl service scan` to scan for bad blocks on a drive:

```text
pxctl service scan --help
```

```output
scan for bad blocks

Usage:
  pxctl service scan [flags]
  pxctl service scan [command]

Available Commands:
  cancel      cancel running scan
  pause       pause running scan
  resume      resume paused scan
  schedule    examine or set schedule
  start       start scan
  status      scan status

Flags:
  -h, --help   help for scan

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

## Delete all Portworx related data

With `pxctl service node-wipe`, you can delete all data related to Portworx from the node. It will also wipe the storage device that was provided to Portworx. This command can be run only when Portworx is stopped on the node. Run this command if a node needs to be re-initialized.

{{<info>}}
This is a disruptive command and could lead to data loss. Please use caution.
{{</info>}}

```text
pxctl service node-wipe --help
```

```output
Wipes PX configuration data on this node

Usage:
  pxctl service node-wipe [flags]

Aliases:
  node-wipe, nw

Flags:
  -h, --help                     help for node-wipe
  -s, --storage_devices string   comma-separated list of storage devices to be wiped.

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

Here is an example:

```text
pxctl service node-wipe
```

```output
This is a disruptive operation.
It will delete all PX configuration files from this node. Data on the storage disks attached on this node will be irrevocably deleted.
Executing manual log rotation logs...
Failed to set pxd timeout. Wipe command might take more time to finish.
Removed PX footprint from device /dev/sdc.
Wiped node successfully.
```

## Perform pool maintenance tasks

The `pxctl service pool` command allows you to run the following pool maintenance related tasks:

- list the available pools
- update the properties of a pool

You can list the available subcommands with:

```text
pxctl service pool
```

```output
Usage:
  pxctl service pool [flags]
  pxctl service pool [command]

Available Commands:
  cache       Update cache properties on a given pool
  expand      Expand pool
  maintenance Pool maintenance
  rebalance   Rebalance storage pools
  show        Show pools
  update      Update pool properties

Flags:
  -h, --help   help for pool

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

## Update pool properties

You can use the `pxctl service pool update` command to perform the following operations:

- Resize a pool
- Set the IO priority
- Add labels

To see the list of the available subcommands, run:

```text
pxctl service pool update --help
```

```output
Update pool properties

Usage:
  pxctl service pool update [flags]

Examples:
pxctl sv pool update [flags] <pool-id> *pool-id or --uid <pool-uid> flag is mandatory, if both are specified, pool-uid is ignored

Flags:
  -h, --help                 help for update
      --io_priority string   IO Priority (Valid Values: [high medium low]) (default "low")
      --labels string        comma separated name=value pairs (empty value to remove label) (default "NoLabel")
      --reset-last-op        Admin command to reset pool's last operation. Pool resources won't be cleaned up. Use with caution!
      --resize               extend pool to maximum available physical storage
  -u, --uid string           Pool's UID

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

#### Understand the --labels flag behavior

The `--labels` flag allows you to add, remove, and update labels for your storage pools.

##### Add a new label

Enter the `pxctl service pool update` command with the pool ID and the `--labels` flag with a comma separated list of labels you wish to add:

```text
pxctl service pool update 0 --labels  ioprofile=HIGH,media_type=SSD
```

##### Replace a label's value

Enter the `pxctl service pool update` command with the pool ID and the `--labels` flag with a comma separated list of the labels you wish to replace:

```text
pxctl service pool update 0 --labels  media_type=NVME
```

Updating a single label does not affect the other labels' stored values.

##### Delete a label's value

Enter the `pxctl service pool update` command with the pool ID and the `--labels` flag with a comma separated list of the labels you wish to delete containing no value:

```text
pxctl service pool update 0 --labels  ioprofile=,media_type=
```

## Delete a pool from a node

If you have a node with multiple pools and you can’t decommission the entire node, you can delete a pool instead. You may need to do this in one of the following scenarios:

   * If there's a problem with the drives on one of your pools, you can delete the pool and reinitialize it either without the problematic drive, or with a new drive.
   * If you've added more capacity to a pool than you need, you can delete your pool and reinitialize it with fewer drives.

{{<info>}}
**NOTE:**

* If you want to _increase_ the size of a storage pool, use the `pxctl service drive add` command.
* Portworx won’t delete a pool until you have manually drained the pool of any volumes.
{{</info>}}

{{<info>}}
**WARNING:** `pxctl service pool delete` is a destructive operation, and all deleted data is not recoverable. Ensure you've taken proper precautions and understand the impact of this operation before running it.
{{</info>}}

Perform the following steps to delete a pool from a node, and optionally, reinitialize it:

1. Once you have identified which pool you want to delete, list all volumes on that pool:

    ```text
    pxctl volume list --pool-uid <pool-uuid>
    ```

2. For each volume on the node, sequentially reduce its replication factor to `1` to isolate the volumes located in your pool. The following example command reduces the replication factor from `3` to `2`:

    ```text
    pxctl volume ha-update --repl=2 --node <node-id> <volume-name>
    ```

3. Verify that your pool is empty by listing all volumes on the pool again:

    ```text
    pxctl volume list --pool-uid <pool-uuid>
    ```

4. Once your pool is empty, delete your pool:

    ```text
    pxctl service pool delete <pool-id>
    ```

5. (Optional) Reinitialize your pool by entering the `pxctl drive add` command, specifying whatever new and existing drives you want to add:

    ```text
    pxctl service drive add --drive /path/to/drive -o start
    ```


### pxctl service pool show

Show storage pool information

```text
pxctl service pool show
```

```output
PX drive configuration:
Pool ID: 0
        IO Priority:  LOW
        Labels:
        Size: 5.5 TiB
        Status: Online
        Has metadata:  No
        Drives:
        0: /dev/sdb, 2.7 TiB allocated of 2.7 TiB, Online
        1: /dev/sdc, 2.7 TiB allocated of 2.7 TiB, Online
        Cache Drives:
        0:0: /dev/nvme0n1, capacity of 745 GiB, Online
                Status:  Active
                TotalBlocks:  762536
                UsedBlocks:  12
                DirtyBlocks:  0
                ReadHits:  487
                ReadMisses:  42
                WriteHits:  1134
                WriteMisses:  7
                BlockSize:  1048576
                Mode:  writethrough
Journal Device:
        1: /dev/sdg1, STORAGE_MEDIUM_MAGNETIC
Metadata  Device:
        1: /dev/sdg2, STORAGE_MEDIUM_MAGNETIC
```

### pxctl service pool cache

You can use the `pxctl service pool cache command` command to:

* Disable caching on a pool
* Enable caching on a pool
* Force the cache to be flushed
* Check if pool caching is enabled for a pool

Refer to the [Pool caching](/concepts/pool-caching) section for more details.

### pxctl service pool delete

You can use the `pxctl service pool delete` command to delete storage pools which may be misconfigured or otherwise not functioning properly.

```text
pxctl service pool delete --help
```
```output
Delete pool

Note:
  This operation is supported only on on-prem local disks and AWS cloud-drive

Usage:
  pxctl service pool delete [flags]

Examples:
pxctl service pool delete [flags] poolID

Flags:
  -h, --help   help for delete

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

Before you remove a pool, consider the following requirements:

* Your target pool for deletion must be empty and contain no replicas
* If your target pool for deletion is a metadata pool, it must be readable
* You must have more pools on the node than just your target pool for deletion
* You must place your node in maintenance mode to use this command

The following example deletes a storage pool from a node containing 2 storage pools:

```text
pxctl service pool delete 0
```
```output
This will permanently remove storage pool and cannot be undone.
Are you sure you want to proceed ? (Y/N): y
Pool 0 DELETED.
```

{{<info>}}
**NOTE:** New pools created after a pool deletion increment from the last pool ID. A new pool created after this example would have a pool ID of 2
{{</info>}}

## Control volume attachments for a node

Portworx allows you to control volume attachments on a node. You can disable new volume attachments on a node by running the `pxctl service node cordon-attachments`. This operation is called as "cordoning attachments" for a node.

You can re-enable volume attachments by running  `pxctl service node uncordon-attachments`. This operation is called as "uncordoning attachments" from a node.

### Cordon attachments on a node

1. Identify which node you want to cordon attachments from by entering the `pxctl cluster list` command:

   ```
   pxctl cluster list
   ```

   Find the ID of your node in the first column:

   ```output
   Cluster ID: local-ddryeu-20-11-17-01-27-43-px-int
   Cluster UUID: fd4fdefa-0830-4453-9f24-091e20d68ad1
   Status: OK

   Nodes in the cluster:
   ID					SCHEDULER_NODE_NAME		DATA IP		CPU		MEM TOTAL	MEM FREE	CONTAINERS	VERSION			Kernel				OS			STATUS
   1cfdb2a5-da08-4158-9883-7fa9e52c59e0	nathan-docs-root-lasher-3	70.0.55.225	3.193277	6.1 GB		4.5 GB		N/A		2.6.2.0-b5b1d0c		3.10.0-862.3.2.el7.x86_64	CentOS Linux 7 (Core)	Online
   c7181271-c4fc-472a-9e5f-28b223b256a0	nathan-docs-root-lasher-0	70.0.31.114	12.827004	6.1 GB		4.5 GB		N/A		2.6.2.0-562f049		3.10.0-862.3.2.el7.x86_64	CentOS Linux 7 (Core)	Online
   2c7dc008-147a-435d-a1ff-a2a1f157f6be	nathan-docs-root-lasher-2	70.0.59.32	3.109244	6.1 GB		4.5 GB		N/A		2.6.2.0-562f049		3.10.0-862.3.2.el7.x86_64	CentOS Linux 7 (Core)	Online
   ```

2. Enter the `pxctl service node cordon-attachments` command with the `--node string` flag followed by the ID of the node you want to cordon:

   ```text
   pxctl service node cordon-attachments --node <node-ID>
   ```
   ```output
   Volume attachments cordoned on node 1cfdb2a5-da08-4158-9883-7fa9e52c59e0.
   Note: Volumes which are already attached on this node will stay attached.
   To drain existing attachments use: pxctl service node cordon-attachments submit --node 1cfdb2a5-da08-4158-9883-7fa9e52c59e0
   ```

### Uncordon attachments from a node

To re-enable volume attachments on a node, enter the `pxctl service node uncordon-attachments` command with the `--node string` flag followed by the ID of the node you want to remove the cordon for:

```text
pxctl service node uncordon-attachments --node <node-ID>
```
```output
Volume attachments re-enabled on node 1cfdb2a5-da08-4158-9883-7fa9e52c59e0.
```

## Drain volume attachments

If you have a node with volumes attached to it, you can remove them using the `pxctl service node drain-attachments` command. This command executes volume drain operations as a background job, and will delete all the pods that are using the volumes which are attached on this node.

### Start volume attachment drain operations

1. Identify which node you want to drain of sharedv4 volumes by entering the `pxctl cluster list` command:

   ```
   pxctl cluster list
   ```

   Find the ID of your desired node in the first column:

   ```output
   Cluster ID: local-ddryeu-20-11-17-01-27-43-px-int
   Cluster UUID: fd4fdefa-0830-4453-9f24-091e20d68ad1
   Status: OK

   Nodes in the cluster:
   ID					SCHEDULER_NODE_NAME		DATA IP		CPU		MEM TOTAL	MEM FREE	CONTAINERS	VERSION			Kernel				OS			STATUS
   1cfdb2a5-da08-4158-9883-7fa9e52c59e0	nathan-docs-root-lasher-3	70.0.55.225	3.193277	6.1 GB		4.5 GB		N/A		2.6.2.0-b5b1d0c		3.10.0-862.3.2.el7.x86_64	CentOS Linux 7 (Core)	Online
   c7181271-c4fc-472a-9e5f-28b223b256a0	nathan-docs-root-lasher-0	70.0.31.114	12.827004	6.1 GB		4.5 GB		N/A		2.6.2.0-562f049		3.10.0-862.3.2.el7.x86_64	CentOS Linux 7 (Core)	Online
   2c7dc008-147a-435d-a1ff-a2a1f157f6be	nathan-docs-root-lasher-2	70.0.59.32	3.109244	6.1 GB		4.5 GB		N/A		2.6.2.0-562f049		3.10.0-862.3.2.el7.x86_64	CentOS Linux 7 (Core)	Online
   ```

2. enter the submit command:

   ```text
   pxctl service node drain-attachments submit --node <node-id>
   ```
   ```
   Drain volume attachments request: 624258766140697912 submitted successfully for node 1cfdb2a5-da08-4158-9883-7fa9e52c59e0
   For latest status: pxctl service node drain-attachments status --job-id 624258766140697912
   ```

   {{<info>}}
   **NOTE:** The drain command will disable new volume attachments on the node. To re-enable attachments on this node, run the `pxctl service node uncordon-attachments` command.
   {{</info>}}

### List volume attachment drain operations

If you want to a list of drain operations Portworx has performed, enter the `pxctl service node drain-attachments list` command:

```
pxctl service node drain-attachments list
```
```
JOB			TYPE			STATE	CREATE TIME
624258766140697912	DRAIN_ATTACHMENTS	DONE	2020-11-23T22:49:45.769448246Z
```

### Monitor volume attachment drain operations

To monitor specific jobs, enter the `pxctl service node drain-attachments status` command with `--job-id` flag, followed by the ID of the specific job you want to see the status for:

```
pxctl service node drain-attachments status --job-id 624258766140697912
```
```
Drain Volume Attachments Summary:

NodeID                   : 1cfdb2a5-da08-4158-9883-7fa9e52c59e0
Job ID                   : 624258766140697912
Job State                : DONE
Last updated             : Mon, 23 Nov 2020 22:50:17 UTC
Status                   : all volume attachments removed from this node

Job summary
Total number of volumes attached on this node                   : 0
Total number of volume attachments removed from this node       : 0
Total number of pending volume attachments                      : 0
```
