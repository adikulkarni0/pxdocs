---
title: Upgrade Portworx (or Kubernetes) on an air-gapped cluster
linkTitle: Air-gapped cluster upgrade
weight: 400
logo: /logos/other.png
keywords: upgrade, on-premises, kubernetes, k8s, air gapped
description: How to upgrade Portworx or Kubernetes version in an air-gapped cluster
noicon: true
aliases:
    - /portworx-install-with-kubernetes/on-premises/airgapped/airgap-upgrade/
    - /install-portworx/on-premises/airgapped/airgap-upgrade/
---

During installation on an internet-connected Kubernetes cluster, Portworx fetches the resources necessary for installation from the internet automatically. However, while installing Portworx on an air-gapped cluster, you would have to perform an extra step to pre-stage these resources within the air-gapped environment.

Similarly, to upgrade your Portwiorx installation on an air-gapped cluster, you will fetch updated container images, and then pre-stage them within the air-gapped cluster.

Since, Portworx leverages a component of the Kubernetes control plane to make enhanced Kubernetes scheduling decisions [based on storage layout](/operations/operate-kubernetes/storage-operations/kubernetes-storage-101/hyperconvergence/). Therefore, these steps are also required if your cluster's Kubernetes/control plane is updated as well.


{{<info>}}
**WARNING:** If you do not perform these pre-staging steps before upgrading either Portworx or Kubernetes on an air-gapped cluster, your pods can enter crash loops and cause service disruptions.
{{</info>}}

Follow the instructions on this page to get the updated container images, pre-stage them within the air-gapped cluster, and then proceed to upgrade your Portworx installation.

## Prerequisites

* You must have an existing Portworx Kubernetes cluster that is healthy and operational.
* You should be using the same internal or private container registry as used during the installation.

{{<info>}}
**NOTE:** To check what registry you're currently using, query your existing StorageCluster by running the following command:

```text
STORAGECLUSTER_NAME=$(kubectl -n kube-system get storagecluster -o jsonpath='{.items[0].metadata.name}')
kubectl get stc -n kube-system $STORAGECLUSTER_NAME -o jsonpath='{.spec.image}{"\n"}'
``` 
{{</info>}}

## Get the updated container images 

1. Set an environment variable for the Kubernetes version that you are using:

    ```text 
    KBVER=$(kubectl version --short 2>/dev/null | awk -F'[v+_-]' '/Server Version: / {print $3}')
    ```

2. Set an environment variable to the latest major version of Portworx:

    ```text
    PXVER=<portworx-version>
    ```

    {{<info >}}
**NOTE:** For the latest Portworx version, see [Portworx Release Notes](/release-notes/portworx/).
    {{</info>}}

3. On an internet-connected host, fetch the same air-gapped-install bootstrap script used during installation, for the Kubernetes and Portworx versions that you specified:

    ```text
    curl -o px-ag-install.sh -L "https://install.portworx.com/$PXVER/air-gapped?kbver=$KBVER"
    ```

4. Pull the container images required for the specified versions:

    ```text
    sh px-ag-install.sh pull
    ```

5. Log in to the container registry using the docker command:

    ```text
    docker login <your-private-registry>
    ```

6. Push the container images to the same private container registry that is accessible to your air-gapped nodes.  Do not include `http://` in your private registry path:

    ```text 
    sh px-ag-install.sh push <your-registry-path>
    ```

    For example:
    ```
    sh px-ag-install.sh push myregistry.net:5443
    ```

    For example, to push the new images to a specific repo (consult your StorageCluster definition as per the note in the Prerequisites):

    ```
    sh px-ag-install.sh push myregistry.net:5443/px-images
    ```

### Create a version manifest configmap for Portworx Operator 

1. Download the Portworx version manifest:

    ```text
    curl -o versions "https://install.portworx.com/$PXVER/version?kbver=$KBVER"
    ```

2. Update (deleting/recreating) the `px-versions` configmap from the downloaded version manifest:

    ```text
    kubectl -n kube-system delete configmap px-versions
    kubectl -n kube-system create configmap px-versions --from-file=versions
    ```

## Upgrade Portworx installation

{{<info>}}
**NOTE:** Skip this section if you are upgrading only the Kubernetes control plane version.
{{</info>}}

Follow the steps in the [Upgrade Portworx](/operations/operate-kubernetes/upgrade/upgrade-operator/#upgrade-portworx) section to upgrade the Portworx StorageCluster spec.

