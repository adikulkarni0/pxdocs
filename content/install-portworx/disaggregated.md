---
title: Disaggregated installation
description: Learn how to install Portworx in disaggregated mode.
keywords: disaggregated, storageless, autoscaling
weight: 500
aliases:
    - /portworx-install-with-kubernetes/disaggregated/
---

In a [disaggregated](/cloud-references/deployment-arch/#approach-a-separate-storage-and-compute-clusters) deployment of Portworx, the storage nodes are separate from the compute (or _storageless_) nodes. Based on the environment in which you are deploying Portworx, the installation instructions for disaggregated mode change.

## Cloud based autoscaling environments

In a cloud autoscaling environment, Portworx recommends using two separate node groups—one for storage nodes and the other for compute (storageless) nodes. This architecture enables you to scale up and scale down your compute cluster based on your application needs while keeping the storage node group constant.

### Prepare your nodes

Before you deploy Portworx, you need to designate your nodes as storage and storageless (compute). Portworx looks for the following labels on the Kubernetes nodes to identify them as a part the storage node group or the storageless node group:

```text
portworx.io/node-type: storage
portworx.io/node-type: storageless
```

When using a managed Kubernetes cluster in the cloud, such as AKS or EKS, add these label pairs to your node groups when you create the node groups. This way, the cloud will ensure that these labels are always present on the Kubernetes nodes.

### Install Portworx 

Follow the [installation instructions for your cloud provider](/install-portworx/cloud/), making sure to select Operator based deployment.

Once you have generated the StorageCluster spec, you will need to add the following environment variable in the `env` section:

```text
spec:
  env:
  - name: ENABLE_ASG_STORAGE_PARTITIONING
    value: "true"
```

You can also add the environment variables from the [spec generator](http://central.portworx.com) using the Environment Variables dropdown menu.

A sample StorageCluster spec for AWS looks like this:

```text
kind: StorageCluster
apiVersion: core.libopenstorage.org/v1
metadata:
  name: px-cluster-XXXX-XXXX-XXXX-XXXX-XXXX
  namespace: kube-system
  annotations:
    portworx.io/install-source: "https://install.portworx.com/?operator=true&mc=false&kbver=&b=true&kd=type%3Dgp2%2Csize%3D150&s=%22type%3Dgp2%2Csize%3D150%22&c=px-cluster-XXXX-XXXX-XXXX-XXXX-XXXX&stork=true&csi=true&mon=true&tel=false&st=k8s&e=ENABLE_ASG_STORAGE_PARTITIONING%3Dtrue&promop=true"
spec:
  image: portworx/oci-monitor:2.10.1
  imagePullPolicy: Always
  kvdb:
    internal: true
  cloudStorage:
    deviceSpecs:
    - type=gp2,size=150
    kvdbDeviceSpec: type=gp2,size=150
  secretsProvider: k8s
  stork:
    enabled: true
    args:
      webhook-controller: "false"
  autopilot:
    enabled: true
  monitoring:
    prometheus:
      enabled: true
      exportMetrics: true
  featureGates:
    CSI: "true"
  env:
  - name: "ENABLE_ASG_STORAGE_PARTITIONING"
    value: "true"
```

Once this environment variable is set, Portworx will run the nodes which are labeled as `portworx.io/node-type=storage` as storage nodes, and it will run the nodes labeled as `portworx.io/node-type=storageless` as storageless nodes.


## On-premises environments

In an on-premises environment, you can use a disaggregated deployment model when you have a heterogeneous cluster configuration, where some of your nodes have storage disks while the rest of the nodes are compute only with no disks configured. 

### Preparing your nodes

Before you deploy Portworx, you need to designate your nodes as storage and storageless (compute). Portworx looks for the following labels on the Kubernetes nodes to identify them as a part the storage node group or the compute (storageless) node group:

```text
portworx.io/node-type: storage
portworx.io/node-type: storageless
```

### Installing Portworx

Follow the [installation instructions for your on-premises environment](/install-portworx/on-premises/other/operator/), making sure to select Operator based deployment.

Once you have generated the StorageCluster spec, you will need to create two separate node sections in it to define the device settings for the storage and storageless (compute) nodes.

Here is a sample StorageCluster spec that uses node-specific overrides:

```text
apiVersion: core.libopenstorage.org/v1
kind: StorageCluster
metadata:
  name: portworx
  namespace: kube-system
spec:
  image: portworx/oci-monitor:2.10.1
  storage:
    devices:
    - /dev/sda
    - /dev/sdb
  nodes:
  - selector:
      labelSelector:
        matchLabels:
          portworx.io/node-type: "storage"
    storage:
      devices:
      - /dev/nvme1
      - /dev/nvme2
  - selector:
      labelSelector:
        matchLabels:
          portworx.io/node-type: "storageless"
    storage:
      devices: []
```

In this example, Portworx on the nodes labeled as `portworx.io/node-type=storage` expects two disks, `/dev/nvme1` and `/dev/nvme2`, and it will run them as storage nodes. On the other hand, Portworx on the nodes labeled as `portworx.io/node-type=storageless` will ignore any disks that might be found on the node and run as storageless nodes.