---
title: Install Portworx on Elastic Kubernetes Service using EKS Blueprints
linkTitle: Install with creation of new cluster
keywords: Install, on cloud, EKS, Elastic Kubernetes Service, AWS, Amazon Web Services, Kubernetes, k8s, EKS Blueprint
description: Install Portworx on an AWS EKS using EKS Blueprints.
weight: 100
disablenext: true
---

Follow the instructions on this page to install Portworx on EKS using EKS Blueprints and the Kubernetes add-on module. In this method, an AWS EKS cluster is created automatically using the example libraries and then Portworx is deployed.
 


## Prerequisites

* You must have the following installed and configured on your client:
	   * Amazon [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
	   * [kubectl](https://kubernetes.io/docs/tasks/tools/)
	   * Hashicorp [Terraform CLI](https://learn.hashicorp.com/tutorials/terraform/install-cli)

## Install Portworx 


The following instructions use the example libraries available within the Terraform EKS Blueprints repository to create an EKS cluster and install Portworx.

### Configure your setup 

1. Clone the Terraform EKS Blueprints repository:
    
    ```text
    git clone https://github.com/portworx/terraform-eksblueprints-portworx-addon.git
    ```

2. Initialize the Terraform module:

    ```text
    cd blueprint/portworx_with_iam_policy
    terraform init
    ```

3. (Optional) Modify the values of variables in the `main.tf` file, such as name, region, managed_node_groups configurations to set up the cluster according to your requirements. Refer to the [Portworx configuration table](https://github.com/portworx/terraform-eksblueprints-portworx-addon/tree/main/blueprint/portworx_with_iam_policy#portworx-configuration) for more options.


### Apply the deployment with Terraform.

1. Deploy the Amazon Virtual Private Cloud (VPC): 

    ```text 
    terraform apply -target="module.vpc"
    ```
2. Deploy the custom IAM policy for providing necessary permissions to Portworx:

    ```text
    terraform apply -target="aws_iam_policy.portworx_eksblueprint_volume_access"
    ```
    {{<info>}}**NOTE:** Refer to [this page](https://github.com/aws-ia/terraform-aws-eks-blueprints/blob/main/docs/add-ons/portworx.md) for viewing the permissions granted to Portworx through the IAM policy.{{</info>}}

3. Deploy the EKS cluster: 

    ```text
    terraform apply -target="module.eks_blueprints"
    ```

4. Deploy the EKS Blueprints add-ons:

    ```text
    terraform apply -target="module.eks_blueprints_kubernetes_addons"
    ```

{{< content "shared/post-installation.md" >}}




