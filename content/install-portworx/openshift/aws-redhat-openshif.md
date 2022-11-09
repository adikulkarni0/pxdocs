---
title: Install Portworx on Red Hat OpenShift Services on AWS 
linkTitle: AWS Red Hat OpenShift
weight: 210
keywords: Install, on-premises, OpenShift, kubernetes, k8s,ROSA, AWS, OpenShift
description: Install Portworx on Red Hat OpenShift Service on AWS
---

## Prerequisites

* You must have a Red Hat OpenShift Services on AWS (ROSA) cluster deployed on infrastructure that meets the [minimum requirements](/install-portworx/prerequisites) for Portworx
* Your cluster must be running OpenShift 4 or higher
* Your instance size must be at least m5.xlarge with 3 compute nodes and have 3 availability zones
* Your cluster must meet [AWS prerequisites for ROSA](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa_getting_started_iam/rosa-aws-prereqs.html)
* Ensure that OCP service is enabled from your AWS console
* AWS CLI [installed](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install) and [configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#:~:text=settings%20and%20precedence-,Quick%20configuration%20with%20aws%20configure,-For%20general%20use)
* ROSA CLI [installed](https://console.redhat.com/openshift/downloads) and [configured](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa_getting_started_iam/rosa-installing-rosa.html)
* Ensure that any underlying nodes used for Portworx in OCP have Secure Boot disabled



## Configure your environment
 
Follow the instructions in this section to setup your environment before deploying Portworx on a Red Hat OpenShift Services on AWS (ROSA) cluster.

### Create an IAM policy 

Follow these instructions from your AWS IAM console to grant the required permissions to Portworx:

1. From the **IAM** page, click **Roles** in the left pane.
2. On the **Roles** page, type your cluster name in the search bar and press enter. Click your cluster's worker role from the search results.
3. From the worker role summary page, click **Add permissions** on the **Permissions** subpage, and then click **Create inline policy** from the dropdown menu.

2. Copy and paste the following into the **JSON** tab text-box and click **Review policy**:

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
3. Provide **Name** and click **Create policy**.
  Once your policy is successfully created for your cluster's worker role, it will be listed in the **Permissions policies** section.

### Open ports for worker nodes

Perform the following to add the inbound rules so that the AWS EC2 instance uses your specified security groups to control the incoming traffic.


2. From the **EC2** page of your AWS console, click **Security Groups**, under **Network & Security**, in the left pane.

3. On the **Security Groups** page, type your ROSA cluster name in the search bar and press enter. You will see a list of security groups associated with your cluster. Click the link under **Security group ID** of your cluster's worker security group:

    ![Security group](/img/rosa/rosa-security-grp.png)

4. From your security group page, click **Actions** in the upper-right corner, and choose **Edit inbound rules** from the dropdown menu.

5. Click **Add Rule** at the bottom of the screen to add each of the following rules:

  * Allow inbound Custom TCP traffic with Protocol: TCP on ports 17001 - 17022
  * Allow inbound Custom TCP traffic with Protocol: TCP on port 20048
  * Allow inbound Custom TCP traffic with Protocol: TCP on port 111
  * Allow inbound Custom UDP traffic with Protocol: UDP on port 17002
  * Allow inbound NFS traffic with Protocol: TCP on port 2049

    Make sure to specify the **security group ID** of the same worker security group that is mentioned in step 2.

6. Click **Save rule**.

## Install Portworx 

Follow the instructions in this section to deploy Portworx.


### Generate Portworx spec

1. Navigate to [PX-Central](https://central.portworx.com/) to generate an installation spec. 

2. Click **Continue** with Portworx Enterprise option:
  ![PX option screen](/img/pxcentral-install.png)

3. Choose an appropriate license for your requirement and click **Continue**:
  ![PX license screen](/img/pxcentral-license.png)

4. On the **Basic** tab, ensure that the **Use the Portworx Operator** option is selected and choose at least version 2.12 from the **Portworx Version** dropdown. Choose **Built-in** ETCD if you have no external ETCD cluster, and click **Next**.

5. On the **Storage** page, choose **Cloud** as your environment, select **AWS** as your cloud platform, and then click **Next**.

6. Choose your network and click **Next**.

7. On the **Customize** page, select **ROSA Redhat Openshift AWS**, and then click **Finish** to generate a Portworx spec.

### Log in to OpenShift UI 

Log in to the OpenShift console by following the quick access instructions on the [Accessing your cluster quickly](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa-sts-accessing-cluster.html#rosa-sts-accessing-cluster) page in the Red Hat OpenShift Service on AWS documentation.


### Install Portworx Operator using the OpenShift UI

1. From your OpenShift console, select **OperatorHub** in the left pane.

2. On the **OperatorHub** page, search for Portworx and select the **Portworx Enterprise** or **Portworx Essential** card: 

    ![PX-operator from OperatorHub](/img/aro/oc-px-search-catalog.png)

3. Click **Install**:

    ![select catalog](/img/openshift-vsphere/image17-2.png)

4. The Portworx Operator begins to install and takes you to the **Install Operator** page. On this page, select the **A specific namespace on the cluster** option for **Installation mode**. Select the **Create Project** option from the **Installed Namespace** dropdown:

    ![Portworx namespace](/img/openshift-vsphere/portworx-namespace.png)

6. On the **Create Project** window, enter the name as `portworx` and click **Create** to create a namespace called **portworx**.

6. Click **Install** to install Portworx Operator in the `portworx` namespace. 


### Apply Portworx spec using OpenShift UI


1. Once the Operator is installed successfully, create a StorageCluster object from the same page by clicking **Create StorageCluster**:

    ![Portworx Operator](/img/openshift-vsphere/px-storagecluster.png)

3. On the **Create StorageCluster** page, choose **YAML view** to configure a StorageCluster.

4. Copy and paste the above Portworx spec into the text-editor, and click **Create** to deploy Portworx:

    ![Install Portworx from OpenShift Console](/img/rosa/os-storage-cluster.png)

4. Verify that Portworx has deployed successfully by navigating to the **Storage Cluster** tab of the Installed Operators page. Once Portworx has been fully deployed, the status will show as Online:

    ![Portworx status](/img/rosa/portworx-installed.png)


{{< content "shared/post-installation-ocp.md" >}}
  

