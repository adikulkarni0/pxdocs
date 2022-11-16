---
title: Install Portworx on EKS air-gapped cluster 
linkTitle: Air-gapped on EKS
weight: 200
keywords: Install, on-premises, kubernetes, k8s, air gapped, EKS
description: How to install Portworx on EKS air-gapped Kubernetes cluster
noicon: true
disableprevnext: true
---

Follow the instructions on this page to deploy Portworx and its required packages on an EKS air-gapped cluster using your private registry location.

## Prerequisites

* You must have an AWS EKS cluster that meets the Portworx [prerequisites](/install-portworx/prerequisites/).
* You must use one of the following disk types: 
    * GP2
    * GP3
    * IO1
* Recommended disk sizes: 
    * GP2: 150 (GB) size disk is needed as the minimum IOP requirement when running on AWS
    * GP3 specify IOPS required from EBS volume and specify throughput for EBS volume
    * IO1 specify IOPS required from EBS volume
* For production environments {{< companyName >}} recommends 3 Availability Zones (AZs).
* {{< companyName >}} recommends you to set Max storage nodes per availability zone.
* A container registry accessible from the nodes on which Portworx will be deployed. In the below example, AWS Elastic Container Registry (ECR) is being used.
* If using ECR, you must have [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed and configured on your client machine.
* An AWS user-account identity that can push or pull images to the given container registry. 

## Configure your environment 

Follow the steps in this section to configure your EKS environment before installing Portworx.

### Create an IAM policy

Provide permissions for all instances in the autoscaling cluster by creating an IAM policy.

Perform the following steps from your AWS console:

1. Navigate to the [IAM page on your AWS console](https://console.aws.amazon.com/iam/), then select **Policies** under the **Identity and Access Management (IAM)** sidebar section and select the **Create Policy** button in the upper right corner:

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

    {{<info>}}**NOTE:** These are the minimum permissions needed for storage operations for a Portworx cluster. For other IAM permissions required for other Portworx operations (such as backup and DR functionality), see the [credentials reference](/reference/cli/credentials/#create-credentials-on-aws-using-iam) section.
    {{</info>}}

3. Name and create the policy:

    ![Create policy](/img/aws-eks/image9.png)

4. In the **Roles** section, search and select your node group **NodeInstanceRole** using your cluster name. The following example shows **eksctl-victorpeksdemo2-nodegroup-NodeInstanceRole-M9QTT58HQ9Z** as the nodegroup Instance Role:

    ![Search for your policy](/img/aws-eks/Step5-select-nodedegroup-NodeInstanceRole-using-cluster-name.png)

    {{<info>}}**NOTE:** If there are more than one nodegroup `NodeInstanceRole` for your cluster, attach the policy to those `NodeInstanceRole`s as well.
    {{</info>}}

5. Attach the previously created policy by selecting **Attach policies** from the **Add permissions** dropdown on the right side of the screen:
    
    ![Attach your policy](/img/aws-eks/Step6-Click-NodeInstanaceRole-then-attach-the-previously-created-policy.png)

6. Under **Other permissions policies**, search for your policy name. Select your policy name and select the **Attach policies** button to attach it.

    The policy you attached will appear under **Permissions policies**:

    ![Confirm your policy is added](/img/aws-eks/Step7-Verify-that-the-policy-is-added.png)


    {{<info>}}**NOTE:** Copy the ARN for the newly created policy. You will need to specify this value for your `NodeGroup` (either within the AWS create-EKS/nodegroup section if using the AWS console, or in a configuration file if using the `eksctl`) for all nodes in your AWS cluster.{{</info>}}



### Get Portworx container images 

1. Set an environment variable for the Kubernetes version you are using:

    ```text 
    KBVER=$(kubectl version --short | awk -F'[v+_-]' '/Server Version: / {print $3}')
    ```

2. Set an environment variable to the latest major Portworx version:

     ```text
    PXVER=<portworx-version>
     ```

2. On an internet-connected host, download the air-gapped-install bootstrap script for the Kubernetes and Portworx versions that you specified: 

    ```text
    curl -o px-ag-install.sh -L "https://install.portworx.com/$PXVER/air-gapped?kbver=$KBVER"
    ```

3. Pull the container images required for the specified versions:

    ```text
    sh px-ag-install.sh pull
    ```

### Set your container registry 

In order to make the Portworx container images available in your air-gapped cluster, you need to have a container registry that the nodes can access.  In the AWS environment, you can use the Elastic Container Registry (ECR) service. ECR is a repository for a _single_ image, whereas Portworx consists of multiple images. Therefore, you must create a separate ECR repository for each image.  

1. Log in to docker:

    ```text
    aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin XXXXXXXXXXXX.dkr.ecr.us-west-2.amazonaws.com
    ```

2.  Run the following command to create an ECR repository for each image.  The following command will create repositories in the `us-west-2` region, which are differentiated by the `pxmirror` prefix. You must use the region where you are deploying your Portworx cluster in the following command:

    ```text
    for images in $(curl -fsSL install.portworx.com/$PXVER/air-gapped | awk -F / '/^IMAGES="$IMAGES /{print $NF}' | cut -d: -f1); do aws ecr create-repository --repository-name pxmirror/$images --image-scanning-configuration scanOnPush=true --region us-west-2; done
    ```

    You need to press **q** after the creation of each repository until the command is completely executed.

2.  Create the Kubernetes secret to pull the images in the same region:

     ```text
     kubectl create secret docker-registry ecr-pxmirror --docker-server XXXXXXXXXXXX.dkr.ecr.us-west-2.amazonaws.com --docker-username=AWS --docker-password=$(aws ecr get-login-password --region us-west-2) -n kube-system
     ```
    The registry used in above command is:`XXXXXXXXXXXX.dkr.ecr.us-west-2.amazonaws.com/pxmirror`, where `XXXXXXXXXXXX` corresponds to your AWS Account ID number.

    {{<info>}}**NOTE:** Skip the above steps if you are not using the ECR registry.{{</info>}}

3. Push the container images to a private registry that is accessible to your air-gapped nodes. Do not include `http://` in your private registry path:

    ```text 
    sh px-ag-install.sh push XXXXXXXXXXXX.dkr.ecr.us-west-2.amazonaws.com/pxmirror
    ```

### Create a version manifest configmap for Portworx Operator 

1. Download the Portworx version manifest:

    ```text
    curl -o versions "https://install.portworx.com/$PXVER/version?kbver=$KBVER"
    ```
2. Create a configmap from the downloaded version manifest:

    ```text
    kubectl -n kube-system create configmap px-versions --from-file=versions
    ```

## Install Portworx

 Follow the instructions in this section to deploy Portworx.

### Generate specs

1. Navigate to the [PX-Central](https://central.portworx.com/).

2. Select **{{< pxEnterprise >}}** from the product catalog.

3. On the **Product Line** page, choose any option depending on which license you intend to use, then select **Continue** to start the spec generator.

4. On the **Basic** page, ensure that the **Use the Portworx Operator** option is selected. For **Portworx version**, select the same version from the dropdown that you have set in the previous section. If you choose version 2.12, change the namespace to _kube-system_ in the **Namespace** field. Select **Built-in ETCD** and click **Next**.

5. Select **Cloud** as your environment and then select **AWS** as your cloud platform. Keep the recommended default values for the **Configure storage devices** section and click **Next**.

6. Choose your network and click **Next**.

7. On the **Customize** page, for the **Are you running on either of these?** option, select **Amazon Elastic Container Service for Kubernetes (EKS)**. Provide your internal registry path and the details for how to connect to your private registry in **Registry And Image Settings**.

 Click the **Finish** button to create the specs.


### Apply specs

Apply the Operator and StorageCluster specs you generated in the section above by performing the following steps:

1. Deploy the Operator:

    ```text
    kubectl apply -f 'https://install.portworx.com/<PXVER>?comp=pxoperator'
    ```
    ```output
    serviceaccount/portworx-operator created
    podsecuritypolicy.policy/px-operator created
    clusterrole.rbac.authorization.k8s.io/portworx-operator created
    clusterrolebinding.rbac.authorization.k8s.io/portworx-operator created
    deployment.apps/portworx-operator created
    ```

2. Deploy the StorageCluster:

    ```text
    kubectl apply -f 'https://install.portworx.com/<PXVER>?operator=true&mc=false&kbver=&b=true&c=px-cluster-0d8dad46-f9fd-4945-b4ac-8dfd338e915b&stork=true&csi=true&mon=true&tel=false&st=k8s&reg=XXXXXXXXXXXX.dkr.ecr.us-west-2.amazonaws.com&rsec=ecr-pxmirror&promop=true'
    ```
    ```output
    storagecluster.core.libopenstorage.org/px-cluster-0d8dad46-f9fd-4945-b4ac-8dfd338e915b created
    ```
 
{{< content "shared/post-installation.md" >}}

