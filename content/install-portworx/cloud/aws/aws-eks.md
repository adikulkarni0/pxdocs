---
title: Install Portworx on Elastic Kubernetes Service and EKS-D
linkTitle: Elastic Kubernetes Service (EKS)
keywords: Install, on cloud, EKS, Elastic Kubernetes Service, AWS, Amazon Web Services, Kubernetes, k8s
description: Install Portworx on an AWS EKS (Elastic Kubernetes Service) cluster.
weight: 100
noicon: true
series: px-k8s
---

This topic explains how to install Portworx on Amazon Web Services (AWS) Elastic Kubernetes Service (EKS) and EKS-D. 

## Prerequisites

* An AWS EKS cluster that meets the Portworx [prerequisites](/install-portworx/prerequisites/)
* You must use one of the following disk types: 
    * GP2
    * GP3
    * IO1
* Recommended disk sizes: 
    * GP2: 150 (GB) size disk is needed as the minimum IOP requirement when running in AWS
    * GP3 specify IOPS required from EBS volume and specify throughput for EBS volume
    * IO1 specify IOPS required from EBS volume
* For production environments {{< companyName >}} recommends 3 Availability Zones (AZs)
* {{< companyName >}} recommends you set Max storage nodes per availability zone, Portworx will ensure that many storage nodes exist in the zone <!-- what does Max storage nodes mean? Is this a value? -->

{{<info>}}
**NOTE:** 

* You can follow the same procedures explained in this topic to deploy Portworx on AWS Outposts.
* You can also follow the steps in this topic to install Portworx on EKS-D.
* For details on GP2, GP3, and IO1 performance characteristics, refer to the [AWS documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html).
{{</info>}}

## Create an IAM policy

Provide the permissions for all the instances in the autoscaling cluster by creating an IAM role.

Perform the following steps on your AWS Console:

1. Navigate to the IAM page on your AWS console, then select **Policies** under the **Identity and Access Management (IAM)** sidebar section, then select the **Create Policy** button in the upper right corner:

    ![AWS create policy page](/img/aws-eks/image15.png)

2. Choose the **JSON** tab, then paste the following permissions into the editor, providing your own value for `Sid` if applicable:

    ```text 
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "", 
                "Effect": "Allow",
                "Action": [
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
                ],
                "Resource": [
                    "*"
                ]
            }
        ]
    }
    ```

    {{<info>}}
**NOTE:** These are the minimum permissions needed for storage operations for a Portworx cluster. For complete permissions required for all of Portworx storage operations, see the [credentials reference](/reference/cli/credentials/#create-and-configure-credentials).
    {{</info>}}


    ![Permission policy](/img/aws-eks/image3.png)

3. Name and create the policy. <!-- You can also add tags. -->

    ![Create policy](/img/aws-eks/image9.png)

<!-- 
4. Confirm the policy has been created as you intended. You can do this by searching for the policy name on the **Policies** page and expanding the result to reveal the JSON:

    ![Policy review page](/img/aws-eks/Step4-review-the-policy.png)

 -->

5. In the **Roles** section, search for and select your nodegroup `NodeInstanceRole` using your cluster name. The following example shows “eksctl-victorpeksdemo2-nodegroup-NodeInstanceRole-M9QTT58HQ9ZX” as the nodegroup Instance Role:

    ![Search for your policy](/img/aws-eks/Step5-select-nodedegroup-NodeInstanceRole-using-cluster-name.png)

    {{<info>}}
**NOTE:** If there are more than one nodegroup `NodeInstanceRole` for your cluster, attach the policy to those `NodeInstanceRole`s as well.
    {{</info>}}

6. Attach the previously created policy by selecting **Attach policies** from the **Add permissions** dropdown on the right side of the screen:
    
    ![Attach your policy](/img/aws-eks/Step6-Click-NodeInstanaceRole-then-attach-the-previously-created-policy.png)

7. Under **Other permissions policies**, search for your policy name. Select your policy name and select the **Attach policies** button to attach it.

    The policy you attached will appear under **Permissions policies** if successful:

    ![Confirm your policy is added](/img/aws-eks/Step7-Verify-that-the-policy-is-added.png)


## Install Portworx
<!-- I'm hiding this until I can understand why we want to provide two competing methods. 

When you are not using the instance privileges method to grant permissions, you must specify the following AWS environment variables (for the KOPS IAM user) in the StorageCluster spec file:

* AWS_ACCESS_KEY_ID=id
* AWS_SECRET_ACCESS_KEY=key

If generating the StorageCluster spec through the GUI wizard, specify the AWS environment variables in the **Environment Variables** field (refer to step 6 of the next section). If generating the StorageCluster spec through the command line, specify the AWS environment variables using the `e` parameter. -->

<!-- Why is this here? Installing the operator is part of the spec generator's output, isn't it?

### Install Operator

Run the following command to install Operator.

```text
kubectl create -f https://install.portworx.com/?comp=pxoperator
``` 
-->

### Generate specs

To install Portworx with Kubernetes, you must generate Kubernetes manifests that you will deploy in your cluster. 

1. Navigate to the Portworx [spec generator](https://central.portworx.com/specGen/wizard).

1. Select **{{< pxEnterprise >}}** from the product catalog:

    ![Product catalog](/img/install-shared/product-catalog.png)

1. On the **Product Line** page, choose any option depending on which license you intend to use, then select **Continue** to start the spec generator:

    ![Product line](/img/install-shared/product-line.png)


1. Select **Use the Portworx Operator**, specify your desired **Portworx version**, and select **Built-in ETCD**:

    <!-- this screenshot shows built-in, do we want to push people to use that?  -->

    ![Basic tab](/img/install-shared/basic.png)

4. Select **Cloud** and the select **AWS**, then keep the recommend **Storage** defaults:

    ![Select AWS](/img/aws-eks/image2.png)

5. Under **Network**, keep the default values and select **Next**.

    ![Network tab](/img/install-shared/network-default.png)

6. Under the **Customize** Tab, select **Amazon Elastic Container Service for Kubernetes (EKS)** radio button at the **Are you running on either of these?**: 

    <!-- 
    if providing Portworx with the required AWS permissions through **Environment Variables**.  
    -->

    ![Customize screen](/img/aws-eks/image10.png)

    <!-- 
    You can set these values under **Environment Variables** and select **Finish**.

    ![Set Environment Variables](/img/aws-eks/image12.png)
    -->

7. Select the **Finish** button to create the specs:

    ![Apply Operator Spec](/img/aws-eks/image13.png)

### Apply specs

Apply the Operator and StorageCluster specs you generated in the section above using the `kubectl apply` command:

1. Deploy the Operator:

    ```
    kubectl apply -f 'https://install.portworx.com/<version-number>?comp=pxoperator'
    ```
    ```output
    serviceaccount/portworx-operator created
    podsecuritypolicy.policy/px-operator created
    clusterrole.rbac.authorization.k8s.io/portworx-operator created
    clusterrolebinding.rbac.authorization.k8s.io/portworx-operator created
    deployment.apps/portworx-operator created
    ```

2. Deploy the StorageCluster:

    ```
    kubectl apply -f 'https://install.portworx.com/<version-number>?operator=true&mc=false&kbver=&b=true&kd=type%3Dgp2%2Csize%3D150&s=%22type%3Dgp2%2Csize%3D150%22&c=px-cluster-xxxxxx-9b92-568f471c2c26&eks=true&stork=true&csi=true&mon=true&tel=false&st=k8s&e==AWS_ACCESS_KEY_IDXXXXXXXXXXXXXXAWS_SECRET_ACCESS_KEYXXXXXXXXXXXXXXXXXXXXX&promop=true'
    ```
    ```output
    storagecluster.core.libopenstorage.org/px-cluster-0d8dad46-f9fd-4945-b4ac-8dfd338e915b created
    ```
## Monitor Portworx nodes

<!-- these aren't pods, are they? they're storageNode objects. Does this come before or after the pod verification? The notes seem to indicate that it comes after.  -->

1. Enter the following `kubectl get` command and wait until all Portworx nodes show as `Online` in the output:

    ```text
    kubectl -n kube-system get storagenodes -l name=portworx
    ```
    ```output
    NAME                 ID                                     STATUS   VERSION          AGE
    username-k8s1-node0   7652208b-0bdf-4222-ac83-43cf085e764e   Online   2.11.1-3a5f406   4m52s
    username-k8s1-node1   d43b7ddb-9f2f-4dde-81ff-4597de6fdd32   Online   2.11.1-3a5f406   4m52s
    username-k8s1-node2   0eda7c8b-3f6b-4ce2-b393-e2169ffa111c   Online   2.11.1-3a5f406   4m52s
    ```

2. Enter the following `kubectl describe` command with the `NAME` of one of the Portworx nodes you retrieved above to show the current installation status for individual nodes:

    ```text
    kubectl -n kube-system describe storagenode <portworx-node-name>
    ```
    ```output
    ...
    Events:
        Type     Reason                             Age                     From                  Message
        ----     ------                             ----                    ----                  -------
        Normal   PortworxMonitorImagePullInPrgress  7m48s                   portworx, k8s-node-2  Portworx image portworx/px-enterprise:2.10.1.1 pull and extraction in progress
        Warning  NodeStateChange                    5m26s                   portworx, k8s-node-2  Node is not in quorum. Waiting to connect to peer nodes on port 9002.
        Normal   NodeStartSuccess                   5m7s                    portworx, k8s-node-2  PX is ready on this node
    ```

{{<info>}}
**NOTE:** 

* In your output, the image pulled will differ based on your chosen Portworx license type and version.
* For {{< pxEnterprise >}}, the default license activated on the cluster is a 30 day trial that you can convert to a SaaS-based model or a generic fixed license.
* For {{< pxEssentials >}}, your cluster must have internet connectivity so that it can send usage information every 24 hours to renew the license on the cluster. You can convert an Essentials license to either a fixed license or SaaS-based license.
{{</info>}}

{{< content "shared/post-installation.md" >}}

<!-- {{<homelist series2="k8s-postinstall">}} -->
