---
title: Supported Kubernetes Resources
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration
description: What are the supported Kubernetes resources.
weight: 700
---

Asynchronous DR supports the following Kubernetes resources:

* PersistentVolumeClaim
* PersistentVolume
* Deployment
* DeploymentConfig
* StatefulSet
* ConfigMap
* Service
* Secret
* DaemonSet
* ServiceAccount
* Role
* RoleBinding
* ClusterRole
* ClusterRoleBinding
* ImageStream
* Ingress
* Route
* Template
* CronJob
* ResourceQuota
* ReplicaSet
* LimitRange
* PodDisruptionBudget
* NetworkPolicy

{{<info>}}**NOTE:** 

* By default, Stork skips migrating NetworkPolicies which have `CIDR` set. To migrate NetworkPolicies which have `CIDR` set, use the `skipNetworkPolicyCheck: true` flag in the migration object. 
* If a Custom Resource (CR) that you need is not present in this list, you can register a new one. Refer to the [Application Registration](/operations/operate-kubernetes/storage-operations/stateful-applications/application-registration/) document for instructions on how to do this.

{{</info>}}


Asynchronous DR also supports the following CRDs out-of-the-box:

* CassandraDatacenter
* CouchbaseBucket
* CouchbaseCluster
* CouchbaseEphemeralBucket
* CouchbaseMemcachedBucket
* CouchbaseReplication
* CouchbaseUser
* CouchbaseGroup
* CouchbaseRoleBinding
* CouchbaseBackup
* CouchbaseBackupRestore
* IBPCA
* IBPConsole
* IBPOrderer
* IBPPeer
* RedisEnterpriseCluster
* RedisEnterpriseDatabase