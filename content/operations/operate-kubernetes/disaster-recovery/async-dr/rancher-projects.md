---
title: Use Rancher Projects with ClusterPair
linkTitle: Rancher Projects 
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration, Rancher Projects, ClusterPair
description: How to use Rancher Projects with ClusterPAir.
weight: 300
---

{{<info>}}**NOTE**: If you are not using Rancher, skip to the [Schedule your migration](/operations/operate-kubernetes/disaster-recovery/async-dr/schedule-migration) section. {{</info>}}

Rancher has a concept of Projects that allow grouping of resources and Kubernetes namespaces. Depending on the resource and how it is created, Rancher adds the following label or annotation:

```text
field.cattle.io/projectID: <project-short-UUID>
```

The `projectID` uniquely identifies the project, and the annotation or label on the Kubernetes object provides a way to tie a Kubernetes object back to a Rancher project. 

Stork has the capability to map projects from the source cluster to the destination cluster when it migrates Kubernetes resources. It will ensure that the following are transformed when migrating Kubernetes resources to a destination cluster:

* Labels and annotations for projectID `field.cattle.io/projectID` on any Kubernetes resource on the source cluster are transformed to their respective projectIDs on the destination cluster.
* Namespace Selectors on a NetworkPolicy object which refer to the `field.cattle.io/projectID` label will be transformed to their respective projectIDs on the destination cluster.
* Namespace Selectors on a Pod object (Kubernetes version 1.24 or newer) which refer to the `field.cattle.io/projectID` label will be transformed to their respective projectIDs on the destination cluster.

{{<info>}}
**NOTE:**

* Rancher project mappings are supported only with Stork version 2.11.2 or newer.
* All the Rancher projects need to be created on both the source and the destination cluster.
{{</info>}}

While creating the ClusterPair, use the argument `--project-mappings` to indicate which `projectID` on the source cluster maps to a `projectID` on the destination cluster. 
For example:

```text
storkctl generate clusterpair -n <migrationnamespace> <remotecluster> --project-mappings  <projectID-A1>=<projectID-A2>,<projectID-B1>: <projectID-B2>
```
The project mappings are provided as a comma-separate `key=value` pairs. In this example, `projectID-A1` on source cluster maps to `projectID-A2` on the destination cluster, while `projectID-B1` on the source cluster maps to `projectID-B2` on the destination cluster.
