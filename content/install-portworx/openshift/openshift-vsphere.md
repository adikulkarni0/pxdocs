---
title: Install Portworx on OpenShift on vSphere
linkTitle: vSphere
weight: 100
keywords: Install, on-premises, OpenShift, kubernetes, k8s, vSphere
description: How to install Portworx on OpenShift on vSphere
---

This article provides instructions for installing Portworx on OpenShift running on vSphere. To accomplish this, you must:

* Install the Portworx Operator using the Red Hat OperatorHub
* Deploy Portworx using the Operator
* Verify your installation

Once you've successfully installed and verified your Portworx installation, you're ready to start using Portworx. To get started after installation, you may want to perform two common tasks:

* Create a PersistentVolumeClaim
* Set up cluster monitoring

## Prerequsites

* Your cluster must be running OpenShift 4 or higher.
* You must have an OpenShift cluster deployed on infrastructure that meets the [minimum requirements](/install-portworx/prerequisites) for Portworx.
* Ensure that any underlying nodes used for Portworx in OCP have Secure Boot disabled.


## Install the Portworx Operator

Before you can install Portworx on your OpenShift cluster, you must first install the Portworx Operator. Perform the following steps to prepare your OpenShift cluster by installing the Operator.

1. From your OpenShift UI, select **OperatorHub** in the left pane.

2. On the **OperatorHub** page, search for Portworx and select either the **{{< pxEnterprise >}}** or **{{< pxEssentials >}}** Operator:
   
    ![search catalog](/img/openshift-vsphere/image5.png)

3. Click **Install** to install Portworx Operator:

    ![select catalog](/img/openshift-vsphere/image17-2.png)

4. The Portworx Operator begins to install and takes you to the **Install Operator** page. On this page, select **A specific namespace on the cluster** option for **Installation mode**. Choose the **Create Project** option from the **Installed Namespace** dropdown:

    ![Portworx namespace](/img/openshift-vsphere/portworx-namespace.png)

5. On the **Create Project** window, enter the name `portworx` and click **Create** to create a namespace called **portworx**.

5. Click **Install** to deploy Portworx Operator in the `portworx` namespace.

## Deploy Portworx using the Operator

The {{< pxEnterprise >}} Operator takes a custom Kubernetes resource called `StorageCluster` as input. The `StorageCluster` is a representation of your Portworx cluster configuration. Once the `StorageCluster` object is created, the Operator will deploy a Portworx cluster corresponding to the specification in the `StorageCluster` object. The Operator will watch for changes on the `StorageCluster` and update your cluster according to the latest specifications.

For more information about the `StorageCluster` object and how the Operator manages changes, refer to the [StorageCluster](/reference/crd/storage-cluster) article.

### Grant the required cloud permissions

Grant permissions Portworx requires by creating a secret with user credentials:

<!-- This statement used to be generic to all different cloud providers. Here, we're getting the vSphere credentials, but what does Portworx use these credentials for? -->

1. Create a secret using the following template. Retrieve the credentials from your own environment and specify them under the `data` section:

    ```text
    apiVersion: v1
    kind: Secret
    metadata:
        name: px-vsphere-secret
        namespace: portworx
    type: Opaque
    data:
        VSPHERE_USER: <your-vcenter-server-user>
        VSPHERE_PASSWORD: <your-vcenter-server-password>
    ```

    <!-- where do they run this? -->

    * **VSPHERE_USER:** to find your vSphere user, enter the following command: 
        
        ```text
        echo '<vcenter-server-user>' | base64
        ```
    
    * **VSPHERE_PASSWORD:** to find your vSphere password, enter the following command:  
        
        ```text
        echo '<vcenter-server-password>' | base64
        ```

    <!-- I thought this was on openshift? shouldn't it be `oc apply` or something? -->
    Once you've updated the template with your user and password, apply the spec:

    ```text
    oc apply -f <your-spec-name>
    ```


2. Ensure ports 17001-17020 on worker nodes are reachable from the control plane node and other worker nodes.

    <!-- why is this in step 2 and not a prerequisite? do we have commands we can provide? -->


3. If you're running a {{< pxEssentials >}} cluster, then create the following secret with your [Essential Entitlement ID](https://central.portworx.com/profile):

    <!-- It looks like they may need to enter their own entitlement ID value, is that true? -->

    ```text
    oc -n portworx create secret generic px-essential \
        --from-literal=px-essen-user-id=YOUR_ESSENTIAL_ENTITLEMENT_ID \
        --from-literal=px-osb-endpoint='https://pxessentials.portworx.com/osb/billing/v1/register'
    ```

### Generate the StorageCluster spec

To install Portworx with OpenShift, you must generate a `StorageCluster` spec that you will deploy in your cluster.
    
1. Navigate to the Portworx [spec generator](https://central.portworx.com/specGen/wizard).

<!-- above we said "if you're running an Essentials cluster", but here we assume they have to select Enterprise. that seems wrong -->

2. Select **{{< pxEnterprise >}}** from the product catalog:

    ![Portworx Operator](/img/install-shared/product-catalog.png)

3. On the **Product Line** page, Select **{{< pxEnterprise >}}** and click **Continue** to start the spec generator:

    ![Portworx Operator](/img/install-shared/product-line.png)

4. On the **Basic** tab, ensure that the **Use the Portworx Operator** option is selected and choose the latest **Portworx version**. Choose **Built-in ETCD** if you have no external ETCD cluster, and click **Next**.


5. On the **Storage** tab:

    * At the **Select your environment** dialog, select the **Cloud** radio button.
    * At the **Select cloud platform** dialog, select **vSphere**.
    <!-- No advice for type of disk? -->
    * At the bottom pane, enter your **vCenter endpoint**, **vCenter datastore prefix**, and the **Kubernetes Secret Name** you created in step 1 of the [Grant the required cloud permissions](#grant-the-required-cloud-permissions) section:

    ![Portworx Operator](/img/openshift-vsphere/px-central-storage.png)

    Select the **Next** button to continue.

6.  On the **Network** tab, keep the default values and select the **Next** button to continue.


7. On the **Customize** tab, select the **Openshift 4+** radio buttom from the **Are you running either of these?** dialog box. 

    * If you're using a proxy, you can add your details to the **Environment Variables** section:

        ![Portworx Operator](/img/openshift-vsphere/image22.png)

    * If you're using a private container registry, enter your registry location, registry secret, and specify an image pull policy under the **Registry and Image Settings**:

        ![Portworx Operator](/img/openshift-vsphere/image15.png)

    Select the **Finish** button to continue.

9.  Save and download the spec for future reference.


<!-- creating this section in response to some of the comments on the google doc that haven't been answered 
#### Customize the spec

Once you've generated the spec, you can customize it ... 

https://docs.google.com/document/d/1lSB0QIecA5m5zBCK9d0JcAImj5FK8nHDDWDiVIEWlrQ/edit?disco=AAAAakoZwHY

https://docs.google.com/document/d/1lSB0QIecA5m5zBCK9d0JcAImj5FK8nHDDWDiVIEWlrQ/edit?disco=AAAAavk7IWs
-->

### Apply the StorageCluster spec

You can apply the StorageCluster spec in one of two ways:

* Using the OpenShift UI
* Using the CLI

#### Apply the spec using the OpenShift UI

1. Once the Operator is installed successfully, create a StorageCluster object by clicking the **Create StorageCluster** button on the same page:

    ![Portworx Operator](/img/openshift-vsphere/px-storagecluster.png)

3. The spec displayed here represents a very basic default spec. Copy the spec you created with the spec generator and paste it over the default spec in the YAML view, and click **Create**:

    ![Portworx Operator](/img/openshift-vsphere/image21.png)

4. Verify that Portworx has deployed successfully by navigating to the **Storage Cluster** tab of the **Installed Operators** page:

    ![Portworx Operator](/img/openshift-vsphere/image3.png)

    Once Portworx has fully deployed, the status will show as **Online**:

    ![Portworx Operator](/img/openshift-vsphere/portworx-installed.png)

#### Apply the spec using the CLI

If you're not using the OpenShift console, you can create the StorageCluster object using the `oc` command:

1. Apply the generated specs to your cluster with the oc apply command:
    
    ```text
    oc apply -f px-spec.yaml
    ```

2. Using the oc get pods command, monitor the Portworx deployment process. Wait until all Portworx pods show as ready:

    ```text
    oc get pods -o wide -n portworx -l name=portworx
    ```

3. Verify that Portworx is deployed by checking its status with the following commands:
 
    ```text
    PX_POD=$(oc get pods -l name=portworx -n portworx -o jsonpath='{.items[0].metadata.name}')
 
    oc exec $PX_POD -n portworx -- /opt/pwx/bin/pxctl status
    ```

{{< content "shared/post-installation-ocp.md" >}}