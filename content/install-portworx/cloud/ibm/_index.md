---
title: Install Portworx on IBM Cloud
linkTitle: IBM Cloud
logo: /logos/ibm.png
linkTitle: IBM Cloud
weight: 500
keywords: Install, IBM Cloud Kubernetes Service, k8s
description: Deploy Portworx on IBM Cloud Kubernetes Service. See for yourself how easy it is!
aliases:
    - /portworx-install-with-kubernetes/cloud/ibm
---

This document provides instructions for installing Portworx using the Portworx Operator on IBM Cloud Kubernetes Service (IKS) using the IBM catalog. This document provides a default installation configuration which is designed to get you up and running with a typical cluster configuration with the following properties:

* The cluster is located in a single availability zone
* Portworx is installed using an internal KVDB
* Kubernetes has access to the public network and gateway

## Prerequisites

Before you start installing Portworx, ensure that you meet the following minimum prerequisites:

* You must have an IBM Cloud account with admin privileges. Portworx does not support using a service ID. 
* You must have a Kubernetes cluster with at least 3 worker nodes deployed on IBM Cloud, and the cluster must meet both the Portworx [minimum requirements](/install-portworx/prerequisites/) as well as the following requirements: 
  * CPU: 16 
  * Memory: 32 GB 
  * Disk: 100 GB
* You must have ability to provision cloud storage for each worker node. <!-- what does this actually mean? are these IAM permissions? how do they get the ability to provision nodes within IKS? -->

## Install Portworx

1. Navigate to [IBM Cloud](https://cloud.ibm.com/login). From the **Catalog** page, search for and select **Portworx Enterprise**. 

2. From the configuration page, make the following selections: 

  * Under **Select a location**, specify the location in which your Kubernetes cluster is located.
  * Under **Select a pricing plan**, select either **Portworx Enterprise with Disaster Recovery (DR)** or **Enterprise**.
  * Under **Configure your resource**, do the following:
    * Choose a service name or accept the default. 
    * Specify the resource group your Kubernetes cluster is in. 
  * At **IBM Cloud API key**, enter your IBM Cloud API key. 
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

3. Agree to the terms and click **Create** to launch the Portworx cluster. This can take 20 minutes or more.

{{< content "shared/post-installation.md" >}}

## Further reading

* [IBM Redbooks](https://www.redbooks.ibm.com/redbooks/pdfs/sg248440.pdf)
* [IBM Cloud online reference](https://cloud.ibm.com/docs/containers?topic=containers-getting-started)
* [Kubernetes (IBM) reference](https://www.ibm.com/cloud/kubernetes-service/kubernetes-tutorials)
