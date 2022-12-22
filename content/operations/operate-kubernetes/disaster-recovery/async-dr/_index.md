---
title: Asynchronous Disaster Recovery (DR)
linkTitle: Asynchronous DR
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration
description: How to achieve asynchronous DR across Kubernetes clusters using scheduled migrations
weight: 200
hidesections: true
aliases:
    - /portworx-install-with-kubernetes/disaster-recovery/async-dr/
  
---

With asynchronous DR, you can replicate Kubernetes applications and their data between two Kubernetes clusters. Here, a separate {{< pxEnterprise >}} cluster runs under each Kubernetes cluster.

* The source Kubernetes cluster asynchronously backs up apps, configuration and data to the destination Kubernetes cluster.
* Incremental changes from Kubernetes applications and Portworx data are continuously sent to the destination cluster.
* In case the source cluster becomes unavailable, you can activate the applications in the destination cluster.


You can find a list of supported Kubernetes resources in the [Supported Kubernetes](/operations/operate-kubernetes/disaster-recovery/async-dr/supported-kubernetes-resources/) article.


Follow the instructions in the following sections in order to set up an asynchronous DR for your cluster.

{{<info>}}
**NOTE:** The following sections require specifying a `<namespace>`. In your environment, update this to the namespace where Portworx is installed. Typically, Portworx is installed in either the `kube-system` or `portworx` namespace. 
{{</info>}} 

{{<homelist series="setup-AysncDR">}}
