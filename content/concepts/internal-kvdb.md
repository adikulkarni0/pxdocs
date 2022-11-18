---
title: Internal KVDB
description: Explanation on internal KVDB used by Portworx
keywords: internal KVDB, built-in KVDB, Kubernetes, k8s, DCOS, DC/OS, help, how to, explanation
weight: 30000
series: concepts
---

{{<info>}}
**NOTE**: Portworx recommends using the built-in internal KVDB in all cases except when running [Metro DR](https://docs.portworx.com/operations/operate-kubernetes/disaster-recovery/).
{{</info>}}

The built-in internal KVDB is enabled by default when you install Portworx through PX-Central. Portworx automatically deploys the internal KVDB cluster and manages the internal key-value store (KVDB) cluster. The internal KVDB is installed on a set of three nodes in your cluster, and it removes the requirement for an external KVDB such as etcd. The internal KVDB can be used with Kubernetes as well as other schedulers. 

## Use the internal KVDB on Kubernetes

To install Portworx using the built-in internal KVDB, use
[PX-Central](https://central.portworx.com) and select **Built-in** for **ETCD** when creating your Portworx cluster.

## Use the internal KVDB on other schedulers

For all other schedulers, Portworx requires that you provide an external etcd to bootstrap its internal KVDB. Portworx uses this external etcd to discover its own internal KVDB nodes. While installing Portworx, provide the `-b` argument to instruct Portworx to set up an internal KVDB. Along with `-b`, provide the `-k` argument with a list of external etcd endpoints. For more information, see the [install documentation](/install-portworx/install-with-other/) for your orchestrator.

## Storage for the internal KVDB

Internal KVDB metadata needs to be stored on a persistent disk. 

## Metadata drive

Provide a separate drive to Portworx nodes through the `-kvdb_dev` argument.

This metadata drive will be used for storing internal KVDB data as well as Portworx metadata such as journal or timestamp data.

{{<info>}}
**NOTE:** The metadata drive needs to be at least 150Gi in size.
{{</info>}}

## Designate nodes as KVDB nodes

There are two ways in which you can designate a node as a KVDB node:

- You can use custom labels to designate nodes as KVDB nodes through the `PX_METADATA_NODE_LABEL` environment variable.

    For example:

    ```text
    export PX_METADATA_NODE_LABEL="kubernetes.io/role=infra"
    ```

- If you are running Portworx on Kubernetes, you can use the `px/metadata-node=true` label to designate the set of nodes that will run the internal KVDB.

    Use the following command to label nodes in Kubernetes:

    ```text
    kubectl label nodes <list of node-names> px/metadata-node=true
    ```

    Depending upon the labels and their values, a decision will be made:

    - If a node is labeled `px/metadata-node=true`, then it will be a part of the internal KVDB cluster.
    - If node is labeled `px/metadata-node=false`, then it will _not_ be a part of the internal KVDB cluster.
    - If no node labels are found, then all the nodes can potentially run internal KVDB.
    - If an incorrect label is present on the node, such as `px/metadata-node=blah`, then Portworx will not start on that node.
    - If no node is labeled as `px/metadata-node=true`, but one node is labeled as `px/metadata-node=false`, then that node will never be a part of KVDB cluster, but the rest of the nodes are potential internal KVDB nodes.

{{<info>}}
**NOTE:** The `PX_METADATA_NODE_LABEL` environment variable takes precedence over the `px/metadata-node=true` label.
{{</info>}}

## KVDB Node Failover

If a Portworx node which was a part of the internal KVDB cluster goes down for more than 3 minutes, then any other available storage nodes which are not part of the KVDB cluster will try to join it.

This is the set of actions that are taken on a node that tries to join the KVDB cluster:

- The down member is removed from the internal KVDB cluster. (Note: The node will be still part of the Portworx cluster)
- Internal KVDB is started on the new node.
- The new node adds itself to the internal KVDB cluster.

The node label logic holds true even during failover, so if a node has a label set to `px/metadata-node=false`, then it will not try to join the KVDB cluster.

If the offline node comes back up, it will _not_ rejoin the internal KVDB cluster if another node has replaced it.

{{<info>}}
**NOTE:** Only storage nodes are a part of internal KVDB cluster.
{{</info>}}


## Backup

Portworx takes regular internal backups of its key-value space and dumps them into a file on the host. Portworx takes these internal KVDB backups every 2 minutes and places them into the `/var/lib/osd/kvdb_backup` directory on all nodes in your cluster. It also keeps a rolling count of 10 backups at a time within the internal KVDB storage drive.

A typical backup file will look like this:

```text
ls /var/lib/osd/kvdb_backup
```

```output
pwx_kvdb_schedule_153664_2019-02-06T22:30:39-08:00.dump
```

These backup files can be used for recovering the internal KVDB cluster in case of a disaster.

## Recovery

In an event of a cluster failure, such as when all of the internal KVDB nodes and the corresponding drives where their data resides become lost and are unrecoverable, or when quorum is lost and two internal KVDB nodes are lost and are unrecoverable, Portworx can recover its internal KVDB cluster.

Please reach out to [Portworx Technical Support](https://docs.portworx.com/support/) to recover your cluster from such a state.
