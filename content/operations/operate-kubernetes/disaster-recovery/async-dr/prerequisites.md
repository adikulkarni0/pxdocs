---
title: Prerequisites  
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration, prerequisite
description: Prerequisites for Async DR migration.
weight: 100
series: setup-AysncDR
---

* **Secret Store**: [Secret store](/operations/key-management) is configured on both clusters. This will store the credentials for the objectstore.
* **Network Connectivity**: 
  * Portworx: Port 9001 on the destination cluster should be reachable from the source cluster. For OpenShift, port 17001 should be reachable.
  * Kubernetes API End Point: Port 6443, 443, or 8443 should be reachable. Check with your Kubernetes administrator for the respective ports.
* **Stork**: Configured on your source and destination clusters.
* **Version**: The same version of Portworx is installed on the source and destination clusters. 
* **Default Storage Class**: Only one default storage class is configured. Having multiple default storage classes will cause PVC migrations to fail.
* **License**: A DR enabled Portworx license on both the source and destination clusters.
* **Objectstore:** An AWS s3 compatible, AWS s3, GCE Object Storage, or Azure Blob Storage.
* **Cloud Environment**:
  * GKE: The instructions on the [Migration with Stork on GKE](/operations/operate-kubernetes/migration/gke/) page have been implemented on your destination cluster.
  * EKS: The instructions on the [Migration with Stork on EKS](/operations/operate-kubernetes/migration/eks/) page have been implemented on your destination cluster. 


<!-- We need to revisit what Portworx/Kubernetes version to be included in prereq -->