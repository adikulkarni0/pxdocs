---
title: Install and configure OpenShift (OCP) on IBM Cloud
logo: /logos/ibm.png
linkTitle: OCP on IBM
weight: 600
keywords: IBM Cloud OpenShift Service, IBM cloud drive, VPC Gen-2
description: Learn how to install and configure Portworx on IBM with OCP using Cloud Drives.
aliases:
    - /install-portworx/cloud/ibm/ibm-cloud-drives/
hidden: true
---


## OpenShift (OCP) on IBM Cloud

OpenShift can be run on IBM Cloud with a Portworx storage cluster. 

This document provides instructions for installing Portworx using the IBM catalog with OpenShift (OCP) on IBM Cloud. This document provides a default installation configuration which is designed to get you up and running with a typical cluster configuration with the following properties:

* The cluster is located in a single availability zone
* Portworx is installed using an internal KVDB
* Kubernetes has access to the public network and gateway <!-- * While Kubernetes can run on a private network, the instructions in this document use the public network and the gateway. -->

<!-- "intra-cluster" what is this? 
 and focuses on an intra-cluster with a single availability zone. 
 -->

## Prerequisites

Before you start installing Portworx, ensure you meet the following minimum prerequisites:

* You must have an IBM Cloud account with admin privileges. Portworx does not support using a service ID. 
* You must have a OpenShift cluster with at least 3 worker nodes deployed on IBM Cloud, and that cluster must meet both the Portworx [minimum requirements](/install-portworx/prerequisites/) as well as the following requirements: 
  * CPU: 16 
  * Memory: 32 GB 
  * Disk: 100 GB
* You must have ability to provision cloud storage for each worker node. <!-- what does this actually mean? are these IAM permissions? how do they get the ability to provision nodes within IKS? -->

<!--
* The Key-value Database (KVDB) device given above needs to be present only on 3 of your nodes and it should have a unique device name across all the KVDB nodes.  

Can we remove this? we're telling people to use internal KVDB. -->
## Install Portworx

1. Navigate to [IBM Cloud](https://cloud.ibm.com/login). From the **Catalog** page, search for and select **Portworx Enterprise**. 
  <!-- I assume this presents them with a page of selections. I need a screenshot of that page so I know what to call the fields. -->

2. From the configuration page, make the following selections: 

  * Under **Select a location**, specify the location in which your Kubernetes cluster is located.
  * Under **Select a pricing plan**, select either **Portworx Enterprise with Disaster Recovery (DR)** or **Enterprise**.
  * Under **Configure your resource**, do the following:
    * Choose a service name or accept the default. 
    * Specify the resource group your Kubernetes cluster is in. 
  * At **IBM Cloud API key**, enter your IBM Cloud API key. Note, this API key is essential for populating the OpenShift cluster name for the next step.
  * Choose the correct OpenShift cluster for **Kubernetes or OpenShift Cluster name** from the drop down list.
  * At **Portworx cluster name**, enter a valid Portworx cluster name. 
  * At **Cloud drives**, select **Use Cloud Drives** from the drop-down menu. This reveals a number of new fields:
    * For **Number of drives**, select your desired number of cloud drives. 
    * For **Max storage nodes per zone**, specify 3 storage nodes per zone.
    * For **Storage Class name**, retain the default storage class names. All cloud drives should show the same Storage Class name.
    * For **Size**, define your desired disk size in GB.
  * For **Portworx metadata Key-value store**, specify **Portworx KVDB**. This deploys Portworx with an internal KVDB cluster.
  * For **Secret type**, keep **Kubernetes secret**. 
  * Leave **Helm Parameters** blank.
  * Enable CSI.
  * For **Portworx versions**, specify your desired Portworx version.

3. Agree to the terms and click **Create** to launch the Portworx cluster; this can take 20 minutes or more.

{{< content "shared/post-installation.md" >}}

## Further reading

* [IBM Redbooks](https://www.redbooks.ibm.com/redpapers/pdfs/redp5606.pdf)
* [IBM Cloud online reference](https://cloud.ibm.com/docs/containers?topic=containers-getting-started)
* [OpenShift on IBM cloud (IBM) reference](https://www.ibm.com/cloud/openshift)
