---
title: Migrate from DaemonSet to Operator on IBM Cloud
linkTitle: Migrate IBM Cloud
keywords: migrate, daemonset, operator, migration, installation, IBM, Marketplace
description: Learn how to migrate your Portworx cluster from a DaemonSet installation to a Portworx Operator installation on IBM Cloud
weight: 200
---

This page describes how to migrate your Portworx cluster on IBM Cloud from a DaemonSet installation to a Portworx Operator installation.

## Prerequisites

To migrate, you must meet the following prerequisites:

* Kubernetes version 1.16 or newer
* Portworx installed with one DaemonSet; the scenario of two or more DaemonSet installations on one Kubernetes cluster is not supported
* [Helm CLI](https://www.ibm.com/docs/en/cloud-private/3.1.2?topic=guide-installing-helm-cli-helm) installed on your client

## Migrate to Operator

Portworx installation on IBM Cloud is managed through a [Helm](https://github.com/portworx/ibm-helm/tree/master/chart/portworx) chart. In order to migrate your Portworx deployment from a DaemonSet installation to a Portworx Operator installation, follow the instructions in this section.

### Upgrade your Portworx deployment using a Helm chart

1. Add Portworx to your Helm repository:

   ``` text
   helm repo add portworx https://raw.githubusercontent.com/portworx/ibm-helm/master/repo/stable/

   ```
2. Update the Helm repository:

   ```text
   helm repo update
   ```
3. Fetch the latest Portworx installation parameters:

   ```text
   helm get values portworx > /tmp/values.yaml
   ```
4. Apply the following CRDs:
   ```text
   kubectl apply -f "https://raw.githubusercontent.com/portworx/ibm-helm/master/chart/portworx/crds/core_v1_storagecluster_crd.yaml"
   ```
   ```text
   kubectl apply -f "https://raw.githubusercontent.com/portworx/ibm-helm/master/chart/portworx/crds/core_v1_storagenode_crd.yaml"
   ```

4. Upgrade Portworx deployment on your IBM cluster:

   ```text
   helm upgrade portworx portworx/portworx -f /tmp/values.yaml --debug
   ```
5. Wait for the StorageCluster to be created. If you have Portworx DaemonSet installed, the Operator will automatically detect that on startup. The Operator will then create an equivalent StorageCluster object.


### Initiate migration

2. Approve the migration by running the following command:

   ```tet
   kubectl -n kube-system annotate storagecluster --all --overwrite portworx.io/migration-approved='true'
   ```

3. Verify the migration status by running the following command:

   ```text
   kubectl -n kube-system describe storagecluster
   ```

    If the migration completes successfully, you will see the event `Migration completed successfully`. If the migration fails, there is a corresponding event about the failure.

### (Optional) Edit your secrets namespace variable

If your Portworx installation uses IBM Key Protect or HPCS as a secret store, then Portworx cannot use the encryption keys stored there after migration. This is because the value of `PX_SECRETS_NAMESPACE` is set to `kube-system` in the migrated StorageCluster spec. To enable Portworx to use these encryption keys, change the value to `portworx`.

1. In the `kube-system` namespace, modify the `env` field of your migrated StorageCluster spec:

   ```text
   env:
   - name: PX_SECRETS_NAMESPACE
      value: portworx
   ```

2. Restart Portworx on all nodes:

   ```text
   kubectl label nodes --all px/service=restart --overwrite
   ```

3. Verify that Portworx can access IBM Key Protect: 

   ```text
   pxctl secrets ibm list-secrets
   ```
   ```output
   Secret ID
   xxxxx14954651
   xxxxx1441673674
   xxxxx4572819956026
   xxxxx903916660671
   xxxxx2438941251166
   default
   ```
    You will see the list of all encryption keys used by Portworx volumes.

## Reference 

* [Rollback to DaemonSet](/operations/operate-kubernetes/migrate-daemonset/migrate-daemonset-operator/#rollback-to-daemonset)
* [Troubleshooting](/operations/operate-kubernetes/migrate-daemonset/migrate-daemonset-operator/#troubleshooting)

