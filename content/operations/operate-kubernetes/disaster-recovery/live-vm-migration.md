---
title: OpenShift Live VM Migration
linkTitle: Live VM Migration
keywords: Live VM, DR, Disaster Recovery, Kubernetes, k8s, cloud, backup, restore, snapshot, migration, sharedv4, sharedv4 service
hidesections: true
weight: 400
---

Portworx version 2.12.0 and Stork version 2.12.0 support OpenShift Live VM migration. 

To facilitate a Live VM migration, the KubeVirt VMs should be using Portworx [sharedv4 service PVCs](/operations/operate-kubernetes/storage-operations/create-pvcs/create-sharedv4-pvcs/) for persistence.

For information on how to perform a Live VM migration, see [Virtual machine live migration](https://docs.openshift.com/container-platform/4.8/virt/live_migration/virt-live-migration.html) in the OpenShift documentation.