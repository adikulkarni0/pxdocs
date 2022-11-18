---
title: Azure Kubernetes Service (AKS)
keywords: Install, on cloud, AKS, Azure Kubernetes Service, Microsoft, Kubernetes, k8s
description: Learn about installing Portwork on Azure Kubernetes Service.
weight: 100
aliases:
    - /install-portworx/cloud/azure/acs/
    - /install-portworx/cloud/azure/aks/1-prepare
    - /install-portworx/cloud/azure/aks/deploy-px-operator
    - /install-portworx/cloud/azure/aks/deploy-px-daemonset/
---

This topic explains how to install Portworx on Azure Kubernetes Service (AKS). Follow the steps in this topic in order.

{{<info>}}
**NOTE:**  If you have a compute load that can elastically increase or decrease based on workload demand, you might want to learn how to [install Portworx in disaggregated mode](/install-portworx/disaggregated/).
{{</info>}}

## Prerequisites

* An AKS cluster that meets the [Portworx prerequisites](/install-portworx/prerequisites)
* The Azure CLI must be [installed](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
* Disk type supported: Premium or Standard
* Portworx recommends that you set max number of storage nodes. When specified Portworx will ensure the desired number of storage nodes exist in the zone.
* For production environments Portworx, Inc. recommends 3 Availability Zones (AZs) with one node per zone.
* Portworx recommends you set Max storage nodes per availability zone, Portworx will ensure that many storage nodes exist in the zone.
* For existing clusters, name of "AKS cluster Infrastructure Resource Group" or initial Resource Group name used to create the cluster and cluster name.
* If you're using Azure Cloud Drive encryption using your own key, you must have an Azure KeyVault instance created in the same region as the AKS cluster.


## Prepare your AKS platform

To set up the Azure Kubernetes Service (AKS) to use Portworx, follow the steps below. 

<!-- this doesn't directly apply to installation now, so I think we should remove it
For more information on AKS, see [this article](https://docs.microsoft.com/en-us/azure/aks/intro-kubernetes). -->

1. Log in to Azure and save your `az login` subscription ID (`"id"`) for future reference:

    ```text
    az login
    ```
    ```output
    [
        {
            "cloudName": "AzureCloud",
            "homeTenantId": "123abcde-f123-4567-abcd-1234567890ab",
            "id": "123abcde-f123-4567-abcd-1234567890ab",
            "isDefault": true,
            "managedByTenants": [],
            "name": "Example name",
            "state": "Enabled",
            "tenantId": "123abcde-f123-4567-abcd-1234567890ab",
            "user": {
            "name": "username@example.com",
            "type": "user"
            }
        }
    ]
    ```

2. Set the subscription:

    ```text
    az account set --subscription {{<highlight>}}<Your-Azure-Subscription-UUID>{{</highlight>}}
    ```


2. Get the Azure locations using the Azure CLI command:

    ```text
    az account list-locations
    ```
    <!-- Do we need this?  
    Example locations: 

    centralus,eastasia,southeastasia,eastus,eastus2,westus,westus2,northcentralus
    southcentralus,westcentralus,northeurope,westeurope,japaneast,japanwest
    brazilsouth,australiasoutheast,australiaeast,westindia,southindia,centralindia
    canadacentral,canadaeast,uksouth,ukwest,koreacentral,koreasouth

    -->

3. Create an Azure Resource Group by specifying its name and the location in which you will be deploying your AKS cluster:

    ```text
    az group create --name {{<highlight>}}<resource-group-name>{{</highlight>}} --location {{<highlight>}}<location>{{</highlight>}}
    ```

4. [Create the AKS cluster](https://docs.microsoft.com/en-us/azure/aks/) in the above Resource Group using either the Azure CLI or the Azure Portal. If you have already deployed an AKS cluster, then create the Service Principal for the Resource Group in which your AKS cluster is present.

    <!-- ??? create the service principal? We should provide instructions assuming they have not already deployed an AKS cluster -->

5. Create a custom role for Portworx. Enter the subscription ID using the subscription ID value you saved in step 1, also specify a role name:

    ```text
    az role definition create --role-definition '{
    "Name": "{{<highlight>}}<your-role-name>{{</highlight>}}",
    "Description": "",
    "AssignableScopes": [
        "/subscriptions/{{<highlight>}}<your-subscription-id>{{</highlight>}}"
    ],
    "Actions": [
        "Microsoft.ContainerService/managedClusters/agentPools/read",
        "Microsoft.Compute/disks/delete",
        "Microsoft.Compute/disks/write",
        "Microsoft.Compute/disks/read",
        "Microsoft.Compute/virtualMachines/write",
        "Microsoft.Compute/virtualMachines/read",
        "Microsoft.Compute/virtualMachineScaleSets/virtualMachines/write",
        "Microsoft.Compute/virtualMachineScaleSets/virtualMachines/read"
    ],
    "NotActions": [],
    "DataActions": [],
    "NotDataActions": []
}'
    ```

### Create a Service Principal and secret in Azure AD

1. Find the AKS cluster Infrastructure Resource Group, the following command shows the Infrastructure Resource Group for a given cluster name and AKS resource group:

    ```text
    az aks show -n {{<highlight>}}<aks-cluster-name>{{</highlight>}} -g {{<highlight>}}<aks-resource-group>{{</highlight>}} | jq -r '.nodeResourceGroup'
    ```

2. Create a service principal for Portworx custom role and replace the following with your cluster's values:

    * Your AKS cluster name 
    * Your subscription ID 
    * The name of the custom role that you created in the previous section

    ```text
    az ad sp create-for-rbac --role={{<highlight>}}<your-role-name>{{</highlight>}} --scopes="/subscriptions/{{<highlight>}}<your-subscription-id>{{</highlight>}}/resourceGroups/{{<highlight>}}<aks-infrastructure-resource-group>{{</highlight>}}"
    ```
    ```output
    {
      "appId": "123abcde-f123-4567-abcd-1234567890ab",
      "displayName": "azure-cli-2020-10-10-10-10-10",
      "name": "http://azure-cli-2020-10-10-10-10-10",
      "password": "123abcde-f123-4567-abcd-1234567890ab",
      "tenant": "123abcde-f123-4567-abcd-1234567890ab"
    }
    ```

2. Create a secret called `px-azure` to give Portworx access to Azure APIs. Take the following fields from the previous output and use them in the following command:

    * Set `AZURE_TENANT_ID` to the value for `tenant`
    * Set `AZURE_CLIENT_ID` to the value for `appId`
    * Set `AZURE_CLIENT_SECRET` to the value for `password`

    ```text
    kubectl create secret generic -n kube-system px-azure --from-literal=AZURE_TENANT_ID={{<highlight>}}<tenant>{{</highlight>}} \
                                                          --from-literal=AZURE_CLIENT_ID={{<highlight>}}<appId>{{</highlight>}} \
                                                          --from-literal=AZURE_CLIENT_SECRET={{<highlight>}}<password>{{</highlight>}}
    ```
    ```output
    secret/px-azure created
    ```

Now that you've created the secret, you're ready to create the spec and deploy Portworx. The spec generator automatically incorporates the secret that you created, and Portworx will fetch the secret to authenticate. Proceed to the next section to install Portworx.

## Install Portworx on AKS using the Operator

### Generate the specs

To install Portworx with Kubernetes, you must first generate Kubernetes manifests that you will deploy in your cluster. Once the spec is generated you must also install Px Operator.

1. Navigate to [PX-Central](https://central.portworx.com/) and log in, or create an account.
2. Select **Portworx Enterprise** and click Continue.
3. Ensure that **Use the Portworx Operator** is selected, then select your desired Portworx Version. Click **Next** to continue to the next tab. 
4. Select **Cloud** followed by **Azure**. Portworx recommends you to use **Create using a Spec** and select **Premium** volume type.
    {{<info>}}
**NOTE:** 

* Portworx recommends you to set **Max storage nodes per availability zone**. Portworx will ensure that many storage nodes exist in the zone. 
* If you’re using a cloud provider, do not add volumes of different types (Standard and Premium) when configuring storage devices during spec generation. This can cause performance issues and errors.
    {{</info>}}
5. In the Networking section click **Next**.
6. In the Customize section select **Azure Kubernetes Service (AKS)** and click **Finish**.

    {{<info>}}
**NOTE:** To deploy Portworx to an Azure Sovereign cloud, you must go to the **Customize** page and set the value of the `AZURE_ENVIRONMENT` variable. The following example screenshot shows how you can deploy Portworx to the Azure US Government cloud:

![Screenshot showing the AZURE_ENVIRONMENT variable](/img/azure-sovereign-example.png)
    {{</info>}}

### Deploy the Operator

To deploy the Operator, run the command that PX-Central provided, which looks similar to the following:

```text
kubectl apply -f 'https://install.portworx.com/{{<highlight>}}<version-number>{{</highlight>}}?comp=pxoperator'
```
```output
serviceaccount/portworx-operator created
podsecuritypolicy.policy/px-operator created
clusterrole.rbac.authorization.k8s.io/portworx-operator created
clusterrolebinding.rbac.authorization.k8s.io/portworx-operator created
deployment.apps/portworx-operator created
```

### Deploy the StorageCluster

To deploy the StorageCluster, use one of the following methods.

* If you are not enabling cloud drive encryption using your own key, run the command that PX-Central provided, which looks similar to the following:

    ```text
    kubectl apply -f 'https://install.portworx.com/{{<highlight>}}<version-number>{{</highlight>}}?operator=true&mc=false&kbver=&b=true&kd=type%3DPremium_LRS%2Csize%3D150&s=%22type%3DPremium_LRS%2Csize%3D150%22&c=px-cluster-4db155bb-69c7-4db6-957b-3aefe978ab64&aks=true&stork=true&csi=true&mon=true&tel=false&st=k8s&promop=true'
    ```
    ```output
    storagecluster.core.libopenstorage.org/px-cluster-0d8dad46-f9fd-4945-b4ac-8dfd338e915b created
    ```

* If you want to enable Azure cloud drive encryption using your own key, perform the following steps:<br><br>

    1. Download the spec that you generated in PX-Central.

    2. Create a Disk Encryption Set ID by using the instructions on [this page](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-enable-customer-managed-keys-portal) in the Microsoft documentation.

    3. Append the `diskEncryptionSetID` value from the previous step to the StorageCluster spec you downloaded as follows:

        ```text
        cloudStorage:
            deviceSpecs:
            - type=Premium_LRS,size=50,diskEncryptionSetID={{<highlight>}}<disk-encryption-set-id>{{</highlight>}}
        ```

        Where `<disk-encryption-set-id>` is in the following format:

        ```text
        /subscriptions/{{<highlight>}}<subscription>{{</highlight>}}/resourceGroups/{{<highlight>}}<resource-group>{{</highlight>}}/providers/Microsoft.Compute/diskEncryptionSets/{{<highlight>}}<disk-encryption-set-name>{{</highlight>}}
        ```
    4. Apply the modified spec:

        ```text
        kubectl apply -f {{<highlight>}}<spec-file.yaml>{{</highlight>}}
        ```
        ```output
        storagecluster.core.libopenstorage.org/px-cluster-0d8dad46-f9fd-4945-b4ac-8dfd338e915b created
        ```

{{< content "shared/post-installation.md" >}}
