---
layout: page
title: "Operator Release Notes"
description: Release notes for the Portworx Operator.
keywords: portworx, release notes, operator
weight: 35000
series: release-notes
aliases:
    - /reference/release-notes/operator
---

## 1.10.1

Dec 5, 2022

### Updates

* Added support for Kubernetes version 1.25, which includes:

    * Removed `PodSecurityPolicy` when deploying Portworx with Operator.

    * Upgraded the API version of `PodDisruptionBudget` from policy/v1beta1 to policy/v1

* Added a UI option in the spec generator to configure Kubernetes version when you choose to deploy Portworx version 2.12.

* Operator is now deployed without verbose log by default. To enable it, add the `--verbose` argument to the Operator deployment.

* For CSI deployment, the px-csi-ext pods now set Stork as a scheduler in the px-csi-ext deployment spec.

* Operator now chooses `maxStorageNodesPerZone`’s default value to efficiently manage the number of storage nodes in a cluster. For more details, see [Manage the number of storage nodes](/cloud-references/auto-disk-provisioning/manage-storage-nodes/).


## 1.10.0

Oct 24, 2022

### Notes

{{<info>}}
**IMPORTANT:** To enable telemetry for DaemonSet-based Portworx installations, you must migrate to an Operator-based installation, then upgrade to Portworx version 2.12 before enabling Pure1 integration. For more details, see [this document](/operations/operate-kubernetes/troubleshooting/enable-pure1-upgrades/).
{{</info>}}

### Updates

* Pure1 integration has been re-architected to be more robust and use less memory. It is supported on Portworx version 2.12 clusters deployed with Operator version 1.10.  
* To reduce memory usage, added a new argument `disable-cache-for` to disable Kubernetes objects from controller runtime cache. For example,`--disable-cache-for="Event,ConfigMap,Pod,PersistentVolume,PersistentVolumeClaim"`.   
* Operator now blocks Portworx installation if Portworx is uninstalled without a wipe and then reinstalled with a different name.
* For a new installation, Operator now sets the max number of storage nodes per zone, so that the 3 storage nodes in the entire cluster are uniformly spread across zones.

### Bug fixes

* Fixed a bug where DaemonSet migration was failing if the Portworx cluster ID was too long.


## 1.9.1

Sep 8, 2022

### Updates

* Added support for Kubernetes version 1.24: 
    * Added `docker.io` prefix for component images deployed by Operator.
    * To determine Kubernetes master nodes, Operator now uses the `control-plane` node role instead of `master`.

### Bug Fixes

* In Operator 1.9.0, when you enabled the CSI snapshot controller explicitly in the StorageCluster, the `csi-snapshot-controller` sidecar containers might have been removed during an upgrade or restart operation. This issue is fixed in Operator 1.9.1.

## 1.9.0
Aug 1, 2022

### Updates

* Daemonset to Operator migration is now Generally Available. This includes the following features:
 * The ability to perform a dry run of the migration
 * Migration for generic helm chart from Daemonset to the Operator
 * Support for the `OnDelete` migration strategy
 * Support for various configurations such as external KVDB, custom volumes, environment variables, service type, and annotations
* You can now use the [`generic helm chart`](https://github.com/portworx/helm/tree/portworx-operator-v1) to install Portworx with the Operator. Note: Only AWS EKS has been validated for cloud deployments.
* Support for enabling [`pprof`](https://pkg.go.dev/runtime/pprof#hdr-Profiling_a_Go_program) in order to get Portworx Operator container profiles for memory, CPU, and so on.
* The Operator now creates example CSI storage classes.
* The Operator now enables the CSI snapshot controller by default on Kubernetes 1.17 and newer.

### Bug Fixes

* Fixed an issue where KVDB pods were repeatedly created when a pod was in the `evicted` or `outOfPods` status.

### Known Issues

* When you upgrade Operator to version 1.9.0, the snapshot controller containers are removed from `px-csi-ext` deployment when the `installSnapshotController` flag is set to true explicitly in the StorageCluster spec.<br>**Workaround:** To fix this issue, either restart Operator or upgrade to a newer version.


## 1.8.1
June 22, 2022

### Updates

* Added support for Operator to run on IPv6 environment.
* You can now enable CSI topology feature by setting the `.Spec.CSI.Topology.Enabled` flag to `true` in the StorageCluster CRD, the default value is `false`. The feature is only supported on FlashArray direct access volumes.
* Operator now uses custom SecurityContextConstraints `portworx` instead of `privileged` on OpenShift.
* You can now add custom annotations to any service created by Operator.
* You can now configure `ServiceType` on any service created by Operator.

### Bug Fixes

* Fixed pod recreation race condition during OCP upgrade by introducing exponential back-off to pod recreation when the `operator.libopenstorage.org/cordoned-restart-delay-secs` annotation is not set. 
* Fixed the incorrect CSI provisioner arguments when custom image registry path contains ":".


## 1.8.0

Apr 14, 2022

### Updates

* Daemonset to operator migration is in Beta release.
* Added support for passing custom labels to Portworx API service from StorageCluster.
* Operator now enables the Autopilot component to communicate securely using tokens when PX-Security is enabled in the Portworx cluster.
* Added field `preserveFullCustomImageRegistry` in StorageCluster spec to preserve full image path when using custom image registry.
* Operator now retrieves the version manifest through proxy if `PX_HTTP_PROXY` is configured.
* Stork, Stork scheduler, CSI, and PVC controller pods are now deployed with [`topologySpreadConstraints`](https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/) to distribute pod replicas across Kubernetes failure domains.
* Added support for installing health monitoring sidecars from StorageCluster.
* Added support for installing snapshot controller and CRD from StorageCluster.
* The feature gate for CSI is now deprecated and replaced by setting `spec.csi.enabled` in StorageCluster.
* Added support to enable hostPID to Portworx pods using the annotation `portworx.io/host-pid="true"` in StorageCluster.
* Operator now sets `fsGroupPolicy` in the CSIDriver object to `File`. Previously it was not set explicitly, and the default value was `ReadWriteOnceWithFsType`.
* Added `skip-resource` annotation to PX-Security Kubernetes secrets to skip backing them to the cloud.
* Operator now sets the dnsPolicy of Portworx pod to `ClusterFirstWithHostNet` by default.
* When using Cloud Storage, Operator validates that the node groups in StorageCluster use only one common label selector key across all node groups. It also validates that the value matches `spec.cloudStorage.nodePoolLabel` if a is present. If the value is not present, it automatically populates it with the value of the common label selector.

### Bug Fixes

* Fixed Pod Disruption Budget issue blocking Openshift upgrade on Metro DR setup.
* Fixed Stork scheduler's pod anti-affinity by adding the label `name: stork-scheduler` to Stork scheduler deployments.
* When a node level spec specifies a cloud storage configuration, we no longer set the cluster level default storage configuration. Before this fix, the node level cloud storage configuration would be overwritten.
