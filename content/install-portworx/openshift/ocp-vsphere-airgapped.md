---
title: Install Portworx on air-gapped OpenShift Container Platform on vSphere
linkTitle: Air-gapped on vSphere
weight: 300
keywords: Install, on-premises, OpenShift, kubernetes, k8s, vSphere, airgap, airgapped
description: How to install Portworx on air-gapped OpenShift Container Platform on vSphere
noicon: true
disableprevnext: true
---

Follow the instructions on this page to deploy Portworx and its required packages on an air-gapped OpenShift Container Platform cluster on vSphere using the internal OpenShift cluster registry.

## Prerequisites

* You must have an OpenShift Container Platform cluster deployed with infrastructure that meets the [minimum requirements](/start-here-installation/) for Portworx (such as having SecureBoot disabled).
* During this procedure, your OpenShift cluster must temporarily have its internal registry reachable externally to the cluster using the procedure [here](https://docs.openshift.com/container-platform/latest/registry/securing-exposing-registry.html).
* You must also have a Linux host with internet access that has either [Podman](https://docs.podman.io/) or Docker or installed.

## Configure your environment

1. On your internet-connected host, set an environment variable for the Kubernetes version that you are using:

    ```text
    KBVER=$(oc version | awk -F'[v+_-]' '/Kubernetes/ {print $2}')
    ```
2. Set an environment variable to the latest major version of Portworx:

     ```text
    PXVER=<portworx-version>
     ```

3. Download the air-gapped-install bootstrap script for the Kubernetes and Portworx versions that you specified: 

    ```text
    curl -o px-ag-install.sh -L "https://install.portworx.com/$PXVER/air-gapped?kbver=$KBVER"
    ```

4. Pull the container images required for the specified versions:

    ```text
    sh px-ag-install.sh pull
    ```
5. [Authenticate](https://docs.openshift.com/container-platform/latest/registry/securing-exposing-registry.html#:~:text=Log%20in%20with%20podman%20using%20the%20default%20route%3A) to the OpenShift internal registry. <br><br>

    For example:

    ```text
    oc login -u admin -p password https://api.lab.ocp.lan:6443
    ```
    ```output
    Login successful.
    [...]
    Using project "default".
    ```

6. Log in to your registry, substituting `docker` for `podman` if you are not using Podman.

    For example:
    ```text
    podman login -u admin -p $(oc whoami -t) default-route-openshift-image-registry.apps.lab.ocp.lan
    ```
    ```output
    Login Succeeded!
    ```

    {{<info>}}
**NOTE:** If the host you're running Podman from does not have the cluster's certificate authority in its trusted-stores, you will need to pass the `--tls-verify=false` flag to the login command.
    {{</info>}}

7. Push the container images to your internal OpenShift cluster registry.

    For example:
    ```
    sh px-ag-install.sh push default-route-openshift-image-registry.apps.lab.ocp.lan/kube-system
    ```

8. Create a secret for the Operator to use that contains the registry credentials.

    For example:
    ```text
    oc -n kube-system create secret docker-registry px-image-repository \
        --docker-server=image-registry.openshift-image-registry.svc:5000 \
        --docker-username=admin \
        --docker-password=$(oc whoami -t)
    ```
    ```output
    Login Succeeded!
    ```

## Create a version manifest configmap for Portworx Operator

1. Download the Portworx version manifest:

    ```text
    curl -o versions "https://install.portworx.com/$PXVER/version?kbver=$KBVER"
    ```
2. Create a configmap from the downloaded version manifest:

    ```text
    oc -n kube-system create configmap px-versions --from-file=versions
    ```

## Deploy Portworx using the Operator

The {{< pxEnterprise >}} Operator takes a custom Kubernetes resource called `StorageCluster` as input. The `StorageCluster` is a representation of your Portworx cluster configuration. Once the `StorageCluster` object is created, the Operator will deploy a Portworx cluster corresponding to the specification in the `StorageCluster` object. The Operator will watch for changes on the `StorageCluster` and update your cluster according to the latest specifications.

For more information about the `StorageCluster` object and how the Operator manages changes, refer to the [StorageCluster](/reference/crd/storage-cluster) article.

### Create a vCenter user for Portworx

{{< content "shared/vcenter-user-for-portworx.md" >}}

### Provide the vCenter user credentials

In order to grant Portworx the necessary permissions for managing the storage block devices that the storage nodes require, create a secret with user credentials.

1. Create a secret using the credentials from your own environment for the vCenter user that has the required permissions:

    ```text
    oc -n kube-system create secret generic px-vsphere-secret \
        --from-literal='VSPHERE_USER=<yourusername@vsphere.local>' \
        --from-literal='VSPHERE_PASSWORD=<yourpasswordhere>'
    ```

2. If you're running a {{< pxEssentials >}} cluster, then create the following secret with your [Essential Entitlement ID](https://central.portworx.com/profile):

    <!-- It looks like they may need to enter their own entitlement ID value, is that true? -->

    ```text
    oc -n kube-system create secret generic px-essential \
        --from-literal=px-essen-user-id=YOUR_ESSENTIAL_ENTITLEMENT_ID \
        --from-literal=px-osb-endpoint='https://pxessentials.portworx.com/osb/billing/v1/register'
    ```

### Generate a Portworx spec

1. Navigate to [PX-Central](https://central.portworx.com/).

2. Select **{{< pxEnterprise >}}** from the product catalog.

3. On the **Product Line** page, choose any option depending on which license you intend to use, then click **Continue**.

5. Select the **Use the Portworx Operator** and **Built-in ETCD** options. For **Portworx version**, select the same value from the dropdown that you have set as your Portworx version in the previous section. Click **Next**.

6. Under **Select your environment**, choose **Cloud**, then choose **vSphere**. Fill in your values for **vCenter Endpoint**, **vCenter Port**, **vCenter datastore prefix**, and **Kubernetes Secret Name**, then click **Next**.

7. Choose your network options and click **Next**.

8. In the **Customize** tab, for the **Are you running on either of these?** option, select **OpenShift 4+**. In the **Registry And Image Settings** section, provide your internal-cluster address for the internal registry path, and the secret `px-image-repository` created earlier.<br><br>

    **Note**: the details to be specified here for how to fetch the images from your cluster-internal registry are **different** than the parameter specified earlier to the `px-ag-install.sh` script, as they are referenced here from _within_ the cluster.  So per the previous example, what you'd specify here would be: `image-registry.openshift-image-registry.svc:5000/kube-system` (rather than the FQDN specified earlier, where they were externally referenced).<br><br>

    Click the **Finish** button to generate your specs.

    ![Your registry details](/img/private-registery-airgapped.png)
   
### Apply specs

Install the Operator and apply the StorageCluster specs you generated in the section above by performing the following steps:

1. Either install the Portwox Operator from the Openshift Operatorhub as [detailed here](/install-portworx/openshift/operator/1-prepare/), or if your Openshift cluster's Operatorhub does not have the Portworx Operator available, it can be installed using the command:

    ```text
    oc apply -f 'https://install.portworx.com/<PXVER>?comp=pxoperator&reg=image-registry.openshift-image-registry.svc:5000/kube-system'
    ```

2. Deploy the StorageCluster using the command PX-Central provided, replacing `kubectl` with `oc`. The provided command will look similar to the following:

    ```text
    kubectl apply -f '<storagecluster-deployment-URL>'
    ```
    ```output
    storagecluster.core.libopenstorage.org/px-cluster-<randomUUID> created
    ```

3. If you did NOT use the OperatorHub and manually installed the Portworx Operator, you'll then also need to annotate the newly created StorageCluster to make it aware of this being an OpenShift environment:

   ```text
   oc -n kube-system annotate stc $(oc -n kube-system get stc -o jsonpath='{.items[0].metadata.name}') 'portworx.io/is-openshift=true'
   ```

{{< content "shared/post-installation-ocp.md" >}}
