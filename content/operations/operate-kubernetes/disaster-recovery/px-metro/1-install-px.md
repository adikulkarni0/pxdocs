---
title: 1. Prepare your Portworx Cluster
weight: 100
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration
description: Find out how to install a single stretch Portworx cluster across multiple Kubernetes clusters.
aliases:
    - /portworx-install-with-kubernetes/disaster-recovery/px-metro/1-install-px/
---
Follow the instructions on this page to setup a single Portworx cluster that spans across multiple Kubernetes clusters.

## Prerequisites

{{<info>}}
**NOTE:** A maximum replication factor of `2` is supported in a Metro-DR deployment. Using `repl: 3` may result in downtime due to a loss of volume quorum.
{{</info>}}

* **Kubernetes Clusters**: At least two Kubernetes clusters which are part of the same metropolitan area network with a
  maximum network latency of 10ms between them.
* **Version**: A single Portworx cluster of v2.1 or later installed on across Kubernetes clusters and Stork v2.2.4 or
  later on both clusters.
* **Network Connectivity**: Ports 9001 to 9020 open between the two Kubernetes clusters.
* **External Kvdb**: A kvdb like etcd setup outside of the Kubernetes clusters.
* **License**: A DR enabled {{< pxEnterprise >}} license on both the source and destination clusters to use this feature.
* **Witness node requirements**: 4 cores minimum, 8 cores recommended, 4 GB minimum, 8 GB recommended. Also, ensure that [Docker engine](https://docs.docker.com/engine/install/) is installed.

## Installing Portworx

In this mode of operation, a single Portworx cluster will stretch across multiple Kubernetes clusters.

### New Installation

To install Portworx on each of the Kubernetes clusters, you need to generate a separate Portworx Kubernetes
manifest file for each of them using the Portworx spec generator at [PX-Central](https://central.portworx.com).

While generating the spec file for each Kubernetes cluster, specify the same **ClusterID** and **Kvdb Endpoints** in each Kubernetes manifest file to ensure that a single Portworx cluster will stretch across multiple Kubernetes clusters:

* **Cluster ID** (StorageCluster: metadata.name)
* **Kvdb Endpoints** (StorageCluster: spec.kvdb.endpoints)

### Existing Installation

If you already have an existing Kubernetes cluster, you can add another Kubernetes cluster and let its nodes join the same Portworx cluster. 

{{<info>}}
**NOTE**: If your existing Kubernetes cluster uses internal kvdb, then you cannot stretch your Portworx clusters across
multiple Kubernetes cluster. This mode of deployments requires an external kvdb running outside your Kubernetes cluster
{{</info>}}
#### Operator Install

1. If your Kubernetes clusters have exactly the same configuration, use the URL specified in the `install-source` annotation field on the existing Portworx installation to fetch the spec for your new cluster:

  ```text
  apiVersion: core.libopenstorage.org/v1
  kind: StorageCluster
  metadata:
    annotations:
      portworx.io/install-source: "https://install.portworx.com/{{% currentVersion %}}?mc=false&kbver=1.11.9&k=etcd%3Ahttp%3A%2F%2F100.26.199.167%3A2379&s=%22type%3Dgp2%2Csize%3D150%22&c=px-cluster-2f6d696f-a728-46ec-bfbc-dcece1765579&stork=true&lh=true&st=k8s"
  ```
    Otherwise, generate a new spec from [PX-Central](https://central.portworx.com).

2. Specify the same **ClusterID** and **Kvdb Endpoints** as your existing cluster, so that a single Portworx cluster will stretch across multiple Kubernetes clusters:

  * **Cluster ID** (StorageCluster: metadata.name)
  * **Kvdb Endpoints** (StorageCluster: spec.kvdb.endpoints)

  
#### DaemonSet Install

1. If your Kubernetes clusters have exactly the same configuration, use the URL specified by
the `install-source` annotation field on the existing Portworx installation to fetch the spec for your new cluster:

  ```text
  apiVersion: apps/v1
  kind: DaemonSet
  metadata:
    annotations:
      portworx.com/install-source: "https://install.portworx.com/{{% currentVersion %}}?mc=false&kbver=1.11.9&k=etcd%3Ahttp%3A%2F%2F100.26.199.167%3A2379&s=%22type%3Dgp2%2Csize%3D150%22&c=px-cluster-2f6d696f-a728-46ec-bfbc-dcece1765579&stork=true& lh=true&st=k8s"
  ```
  
    Otherwise, generate a new spec from [PX-Central](https://central.portworx.com).

2. Specify the same **ClusterID** and **Kvdb Endpoints** as your existing cluster, so that a single Portworx cluster will stretch across multiple Kubernetes clusters:  

  * **Cluster ID** (Portworx install argument: `-c`)
  * **Kvdb Endpoints** (Portworx install argument: `-k`)
  
### Specifying cluster domains

A cluster domain identifies a subset of nodes from the stretch Portworx cluster that are a part of the same failure
domain. In this case, your Kubernetes clusters are separated across a metropolitan area network, and you can achieve DR across them. So each Kubernetes cluster and its nodes are one cluster domain. This cluster domain information needs to be explicitly specified to Portworx through the `-cluster_domain` install argument.

Once you have generated the Kubernetes manifest file, add the `cluster_domain` argument. You can also edit a running Portworx install and add this new field. The value for `cluster_domain` must be different for each Kubernetes cluster, for example:

* You can name your cluster domains with arbitrary names like **datacenter1** and **datacenter2** identifying in which
  Datacenter your Kubernetes clusters are running.
* You can name them based on your cloud provider's zone names such as **us-east-1a**, **us-east-1b**.

#### Using the Operator

Use the `portworx.io/misc-args` annotation to add a `-cluster_domain` argument with the cluster domain name of your cluster environment. The following example uses a cluster domain of `us-east-1a`:

```text
apiVersion: core.libopenstorage.org/v1
kind: StorageCluster
metadata:
  annotations:
    portworx.io/misc-args: "-cluster_domain us-east-1a"
```

#### Using the DaemonSet

Add the `-cluster_domain` argument in the `args` section of the DaemonSet. The following example uses a cluster domain of `us-east-1a`:

```text
      containers:
        - name: portworx
          image: portworx/oci-monitor:2.1
          imagePullPolicy: Always
          args:
            ["-k", "etcd:http://100.26.199.167:2379,etcd:http://100.26.199.168:2379,etcd:http://100.26.199.169:2379", "-c", "px-dr-cluster", "-cluster_domain", "us-east-1a", "-a", "-secret_type", "k8s",
             "-x", "kubernetes"]

```
{{<info>}}**NOTE:** Additional steps are required when installing on a Tanzu cluster. For instructions, refer to [Installing on a Tanzu cluster](/operations/operate-kubernetes/disaster-recovery/px-metro/tanzu) page.{{</info>}}

### Setup a witness node

To set up a witness node: 

1. Check your Portworx Enterprise version:

  ```text
  kubectl get pods -A -o jsonpath="{.items[*].spec.containers[*].image}" | xargs -n1 | sort -u | grep oci-monitor
  ```

2. Install the witness node with the same Portworx Enterprise version that you retrieved in the previous step, after the Docker engine is up and running:

    ```txt
    chmod +x ./witness-install.sh ./witness-install.sh --cluster-id=px-cluster --etcd="etcd:http://70.0.68.196:2379,etcd:http://70.0.93.183:2379,etcd:http://70.0.68.196:2379" --docker-image=portworx/px-enterprise:<your-px-version>
    ```

3. Set up a single storage-less Portworx node on the designated VM using the [witness script](#appendix).

   {{<info>}}**NOTE:** 

   * Refer to [Upgrade the Portworx OCI bundle](/install-portworx/install-with-other/docker/standalone#upgrade-the-portworx-oci-bundle) section to upgrade witness node.

   * Refer to [Uninstall the Portworx OCI bundle](/install-portworx/install-with-other/docker/standalone#uninstall-the-portworx-oci-bundle) section to uninstall witness node.
   {{</info>}}
### Install storkctl

`storkctl` is a command-line tool for interacting with a set of scheduler extensions.

{{< content "shared/portworx-install-with-kubernetes-disaster-recovery-stork-helper.md" >}}

### Cluster Domains Status

When Stork is deployed along with Portworx in this mode it automatically creates a ClusterDomainStatus object. This can
be used to find out the current state of all the cluster domains. Each Kubernetes cluster where Stork is deployed will
have this object automatically created. You can use the following commands to get the current status:

```text
storkctl get clusterdomainsstatus
```

```output
NAME            ACTIVE                    INACTIVE   CREATED
px-dr-cluster   [us-east-1a us-east-1b]   []         09 Apr 19 17:12 PDT
```

You can also use `kubectl` to inspect the ClusterDomainsStatus object

```text
kubectl describe clusterdomainsstatus -n kube-system
```

```output
Name:         px-dr-cluster
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  stork.libopenstorage.org/v1alpha1
Kind:         ClusterDomainsStatus
Metadata:
  Creation Timestamp:  2019-04-10T00:12:53Z
  Generation:          1
  Resource Version:    62704
  Self Link:           /apis/stork.libopenstorage.org/v1alpha1/clusterdomainsstatuses/px-dr-cluster
  UID:                 62865df5-5b25-11e9-a3ff-12e335b4e36e
Status:
  Active:
    us-east-1b
    us-east-1a
  Inactive:  <nil>

```

Once your Portworx stretch cluster is installed, you can proceed to next step.

## Appendix

Following is the witness script mentioned in the [Setup a witness node](#setup-a-witness-node) section:

```txt
#!/bin/bash

PX_DOCKER_IMAGE=portworx/px-enterprise:2.11.0

function usage()
{
echo "Install Portworx as a witness node"
echo ""
echo "./install-witness.sh"
echo "--help"
echo "--cluster-id=$CLUSTER_ID"
echo "--etcd=$ETCD"
echo "--docker-image=$PX_DOCKER_IMAGE"
echo ""
}

while [ "$1" != "" ]; do
PARAM=`echo $1 | awk -F= '{print $1}'`
VALUE=`echo $1 | awk -F= '{print $2}'`
case $PARAM in
--help)
usage
exit
;;
--cluster-id)
CLUSTER_ID=$VALUE
;;
--docker-image)
PX_DOCKER_IMAGE=$VALUE
;;
--etcd)
ETCD=$VALUE
;;
*)
echo "ERROR: unknown parameter \"$PARAM\""
usage
exit 1
;;
# Once PX is up and running place the witness node in maintenance.
esac
shift
done

sudo docker run --entrypoint /runc-entry-point.sh \
--rm -i --privileged=true \
-v /opt/pwx:/opt/pwx -v /etc/pwx:/etc/pwx \
$PX_DOCKER_IMAGE

/opt/pwx/bin/px-runc install -k $ETCD -c $CLUSTER_ID --cluster_domain witness -a

sudo systemctl daemon-reload
sudo systemctl enable portworx
sudo systemctl start portworx

watch -n 1 --color /opt/pwx/bin/pxctl --color status 
```

In the witness script:

* the `--cluster_domain` argument is set to witness to indicate that this is a witness node and it contributes to quorum.
* the `clusterID` and the `etcd` details need to be the same as they have been provided to the two other Portworx installations done in the Kubernetes clusters.