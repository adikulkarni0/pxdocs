---
title: Install Portworx on IBM Cloud Kubernetes Service using Terraform
linkTitle: Terraform IKS
weight: 510
keywords: Install, IBM Cloud Kubernetes Service, k8s
description: Deploy Portworx on IBM Cloud Kubernetes Service using Terraform module.
---

This document provides instructions for installing Portworx on an IBM Cloud Kubernetes Service (IKS) cluster using the Terraform module. You can use the Terraform module to upgrade an active installation and to automate Portworx deployment across multiple IKS clusters.

## Prerequisites 

* Have an IKS cluster meeting the Portworx [minimum requirements](/install-portworx/prerequisites/).
* Use Terraform version 0.13 or later.
* Ensure that the following utilities are installed and configured on your client:
    * [IBM Cloud Kubernetes Service CLI](https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install)
    * [kubectl](https://kubernetes.io/docs/tasks/tools/)
    * HashiCorp [Terraform CLI](https://learn.hashicorp.com/tutorials/terraform/install-cli)
    * [Helm CLI](https://www.ibm.com/docs/en/cloud-private/3.1.2?topic=guide-installing-helm-cli-helm)
    * The latest versions of the `jq`, `curl`, and `wget` libraries

## Configure your Terraform on IBM Cloud setup

1. Clone the Terraform repository:

    ```text
    git clone https://github.com/portworx/terraform-ibm-portworx-enterprise.git
    ```
2.  Depending on your cluster resources, navigate to the appropriate folder:
 
    * IKS cluster using mounted volumes:
        ```text 
        cd examples/iks-with-attached-drives
        ```
    * IKS cluster using cloud drives:
        ```text
        cd examples/iks-with-cloud-drives
        ```
    * IKS cluster where existing worker nodes are converted to Portworx nodes:
        ```text
        cd examples/iks-worker-node-replace
        ```

### Declare the IBM Cloud resources  

Create a variable file and name it `terraform.tfvars`. In this file, declare the IBM Cloud resources that are required for your Portworx deployment.

* For an IKS cluster using mounted volumes or cloud drives, declare the following:
    
    ```text
    iks_cluster_name="<your_iks-cluster_name>"
    resource_group="<your_resource_group_name>"
    ibmcloud_api_key="<secret_ibm_cloud_key>"
    ```
    
    * `resource_group`: Specify the resource group name where your IKS cluster exists, and the Portworx Enterprise service instance will be created in the same resource group.
    * `iks_cluster_name`: Name of your IKS cluster
    * `ibmcloud_api_key`: IBM Cloud API key associated with your cluster 


* For an IKS cluster where existing worker nodes are converted to Portworx nodes, declare the following:

    ```text
    resource_group   = "<your_resource_group_name>"
    region           = "<region_name>"
    iks_cluster_name = "<your_iks_cluster_name>"
    worker_ids       = ["<worker_id_1>", "<worker_id_2>", "<worker_id_3>"]
    replace_all_workers = false
    ```
  
    * `resource_group`: Specify the resource group name where your IKS cluster exists, and the Portworx Enterprise service instance will be created in the same resource group.
    * `region`: Specify the IBM region where your cluster is provisioned
    * `worker_ids`: List of all worker nodes IDs to be replaced.
    * `iks_cluster_name`: Name of your IKS cluster

For the complete list of variables, refer to the `examples/variables.tf` file in your setup directory.

### Retrieve the kubeconfig file 

To access the IKS cloud services and resources from your client machine, run the following command:

```text
export IC_API_KEY="secret_ibm_cloud_key"
ibmcloud ks cluster config --admin --cluster <cluster_name | cluster_id>
```
## Install Portworx


1. Initialize Terraform:

    ```text
    terraform init
    ```

2. Generate a Terraform execution plan:

    ```text
    terraform plan -out tf.plan
    ```

3. Apply the execution plan:

    ```text
    terraform apply tf.plan
    ```
    
3. Run the following command to deploy Portworx:

    ```text
    terraform output
    ```
    ```output
    associated_iks_cluster_id = "cdkasv9w0oi7m5g5i6qg"
    associated_iks_cluster_name = "schakravorty-px-wdc-b3c.4x16"
    portworx_enterprise_id = "crn:v1:bluemix:public:portworx:us-east:a/b5c47bfad9f99ace9c94c3986bd1fdc9:dcc36d14-3044-4cfe-b9b9-5192d195b094::"
    portworx_enterprise_service_name = "portworx-enterprise-b0d0b748"
    portworx_version_installed = "2.11.4"
    ```

    Wait for a few minutes for the Portworx pods to be created. You will see that the command outputs the details of the Portworx Enterprise service.

{{< content "shared/post-installation.md" >}}