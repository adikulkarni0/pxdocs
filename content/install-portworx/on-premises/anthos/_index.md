---
title: Anthos
linkTitle: Anthos
logo: /logos/anthos.png
weight: 400
keywords: Install, on-premises, anthos, kubernetes, k8s, air gapped
description: How to install Portworx on Anthos
aliases:
    - /portworx-install-with-kubernetes/on-premise/anthos/
---

Portworx has been certified with the following Anthos versions:

* 1.9, including Anthos on bare metal
* 1.10, including Anthos on bare metal
* 1.11, including Anthos on bare metal
* 1.12, including Anthos on bare metal


## Architecture

{{< content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-shared-arch.md" >}}

## Installation

This topic explains how to install Portworx with Kubernetes on Anthos.

{{<info>}}**NOTES:**

* VMware vSphere v6.7u3 or newer is required.
* Run these steps from the anthos admin station or any other machine which has kubectl access to your cluster.
{{</info>}}

{{< content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-install-common.md" >}}

#### Set max storage nodes

Anthos cluster management operations, such as upgrades, recycle cluster nodes by deleting and recreating them. During this process, the cluster momentarily scales up to more nodes than initially installed. For example, a 3-node cluster may increase to a 4-node cluster.

To prevent Portworx from creating storage on these additional nodes, you must cap the number of Portworx nodes that will act as storage nodes. You can do this by setting the `MAX_NUMBER_OF_NODES_PER_ZONE` environment variable according to the following requirements:

* If your Anthos cluster does not have zones configured, this number should be your initial number of cluster nodes
* If your Anthos cluster has zones configured, this number should be initial number of cluster nodes per zone

```text
export MAX_NUMBER_OF_NODES_PER_ZONE=3
```
{{<info>}} **NOTE:** In the command above, 3 is an example. Change this number to suit your cluster.{{</info>}}

#### Install the Portworx Operator

```text
curl -fsL -o px-operator.yaml "https://install.portworx.com/?comp=pxoperator"
```

#### Apply the Portworx Operator spec

```text
kubectl apply -f px-operator.yaml
```

#### Set the Portworx version

Set an environment variable to the latest major version of Portworx:

```text
export PXVER=<portworx-version>
```

#### Generate the spec file

Now generate the storage cluster spec with the following command:

```text
curl -fsL -o px-spec.yaml "https://install.portworx.com/$PXVER?operator=true&gke=true&mz=$MAX_NUMBER_OF_NODES_PER_ZONE&csida=true&c=portworx-demo-cluster&b=true&st=k8s&csi=true&vsp=true&ds=$VSPHERE_DATASTORE_PREFIX&vc=$VSPHERE_VCENTER&s=%22$VSPHERE_DISK_TEMPLATE%22&misc=-rt_opts%20kvdb_failover_timeout_in_mins=25,wait-before-retry-period-in-secs=360"
```

{{< content "shared/portworx-install-with-kubernetes-4-apply-the-spec.md" >}}