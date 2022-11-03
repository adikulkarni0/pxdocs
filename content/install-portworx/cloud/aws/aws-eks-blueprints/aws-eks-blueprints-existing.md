---
title: Install Portworx on Elastic Kubernetes Service using EKS Blueprints on an existing EKS cluster
linkTitle: Install on an existing cluster
keywords: Install, on cloud, EKS, Elastic Kubernetes Service, AWS, Amazon Web Services, Kubernetes, k8s, EKS Blueprint
description: Install Portworx on an AWS EKS using EKS Blueprints.
weight: 110
disablenext: true
---

Follow the instructions on this page to install Portworx on EKS using EKS Blueprints and the Kubernetes add-on module when you have an existing AWS EKS cluster.

{{<info>}}**NOTE:** Portworx recommends creating the cluster through Terraform script using the EKS blueprint module. This will allow you to have a better workflow while using the Kubernetes add-on module with the overall cluster creation.{{</info>}}


## Prerequisites

* An AWS EKS cluster that meets the Portworx [prerequisites](/install-portworx/prerequisites/)
* You must have the following installed and configured on your client:
	   * Amazon [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
	   * [kubectl](https://kubernetes.io/docs/tasks/tools/)
	   * Hashicorp [Terraform CLI](https://learn.hashicorp.com/tutorials/terraform/install-cli)

## Install Portworx 

Follow the instructions in this section to deploy Portworx:

### Configure your environment

1. In your terraform scripts, create a new IAM policy resource to grant necessary permissions to Portworx. Provide it a policy and resource name:

    ```text 
    resource "aws_iam_policy" "<policy-resource-name>" {
        name = "<policy-name>"

        policy = jsonencode({
            Version = "2012-10-17"
            Statement = [
                {
                    Action = [
                        "ec2:AttachVolume",
                        "ec2:ModifyVolume",
                        "ec2:DetachVolume",
                        "ec2:CreateTags",
                        "ec2:CreateVolume",
                        "ec2:DeleteTags",
                        "ec2:DeleteVolume",
                        "ec2:DescribeTags",
                        "ec2:DescribeVolumeAttribute",
                        "ec2:DescribeVolumesModifications",
                        "ec2:DescribeVolumeStatus",
                        "ec2:DescribeVolumes",
                        "ec2:DescribeInstances",
                        "autoscaling:DescribeAutoScalingGroups"
                    ]
                    Effect   = "Allow"
                    Resource = "*"
                },
            ]
        })
    }
    ```
2. Apply the policy:

    ```text 
    terraform apply -target="aws_iam_policy.<policy-resource-name>"
    ```

3. Attach the newly created policy to the node groups in your cluster. If you are using EKS blueprint for your EKS cluster creation, make the addition as shown below:

    ```text
    managed_node_groups = {
        node_group_1 = {
            node_group_name           = "my_node_group_1"
            instance_types            = ["t2.small"]
            min_size                  = 3
            max_size                  = 3
            subnet_ids                = module.vpc.private_subnets

            # Add this line to the code block or add the new policy ARN to the list if it already exists
            additional_iam_policies   = [aws_iam_policy.<policy-resource-name>.arn]

        }
    }
    ```
4. Apply the changes:

    ```text
    terraform apply -target="module.eks_blueprints"
    ```


### Deploy Portworx

To install Portworx, use the Kubernetes add-on module, pass the right cluster configuration values, and set the `enable_portworx` variable to `true`, as shown below:

```text
module "eks_blueprints_kubernetes_addons" {
 source = "github.com/aws-ia/terraform-aws-eks-blueprints//modules/kubernetes-addons"

  eks_cluster_id       = module.eks_blueprints.eks_cluster_id
  eks_cluster_endpoint = module.eks_blueprints.eks_cluster_endpoint
  eks_oidc_provider    = module.eks_blueprints.oidc_provider
  eks_cluster_version  = module.eks_blueprints.eks_cluster_version


  #Add this line to enable Portworx      
  enable_portworx  = true
}
```

Customize Portworx installation when required by passing the cluster configuration parameter as a list of objects as shown below:

```text
  enable_portworx  = true

  portworx_helm_config = {
    set = [
      {
        name  = "clusterName"
        value = "testCluster"
      },
      {
        name  = "imageVersion"
        value = "2.11.1"
      }
  ]}}
```
For more options to customise Portworx installation, see [Portworx configuration](https://github.com/portworx/terraform-eksblueprints-portworx-addon/tree/main/blueprint/portworx_with_iam_policy#portworx-configuration) table.

{{< content "shared/post-installation.md" >}}






