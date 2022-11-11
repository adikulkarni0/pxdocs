---
title: Migrate from DaemonSet to Operator
linkTitle: Migrate from DaemonSet to Operator
keywords: migrate, daemonset, operator, migration, installation
description: Learn how to migrate your Portworx cluster from a DaemonSet installation to a Portworx Operator installation.
weight: 100
aliases:
  - /operations/operate-kubernetes/migrate-daemonset/
---

This page describes how to migrate your Portworx cluster from a DaemonSet installation to a Portworx Operator installation.

## Prerequisites

To migrate, you must meet the following prerequisites:

* Kubernetes version 1.16 or newer
* Portworx installed with one DaemonSet; the scenario of two or more DaemonSet installations on one Kubernetes cluster is not supported

## Migrate to Operator

### Install Portworx Operator

Install the Portworx Operator version 1.9.0 or newer using one of the following methods:

#### Install by spec generator

To install using the spec generator, run the following commands:

```text
curl -o operator.yaml https://install.portworx.com/?comp=pxoperator
```
```text
kubectl apply -f operator.yaml
```

#### Install by Helm chart

Make sure the Helm chart supports the Portworx Operator, then run `helm upgrade`. Refer to the [Helm chart documentation](https://github.com/portworx/helm/tree/portworx-operator-v1/charts/portworx#upgrading-the-chart-from-an-old-chart-with-daemonset) for more details.

{{<info>}}**NOTE:** To migrate your Portworx cluster on IBM Cloud from a DaemonSet installation to a Portworx Operator installation, refer to [this page](/operations/operate-kubernetes/migrate-daemonset/migrate-daemonset-ibm){{</info>}}


### Wait for the StorageCluster to be created

If you have Portworx DaemonSet installed, the Operator will automatically detect that on startup. The Operator will then create an equivalent StorageCluster object.


### Add cloud provider annotation to StorageCluster

If you are using Helm, proceed to the next section.

Otherwise, run the appropriate command for your cloud provider from the following list:

* Pivotal Kubernetes Service (PKS)
   ```text
   kubectl -n kube-system annotate storagecluster --all portworx.io/is-pks="true"
   ```
* IBM Cloud Kubernetes Service (IKS)
   ```text
   kubectl -n kube-system annotate storagecluster --all portworx.io/is-iks="true"
   ```

* Google Kubernetes Engine (GKE)
   ```text
   kubectl -n kube-system annotate storagecluster --all portworx.io/is-gke="true"
   ```
* Azure Kubernetes Service (AKS)
   ```text
   kubectl -n kube-system annotate storagecluster --all portworx.io/is-aks="true"
   ```
* Amazon Elastic Kubernetes Service (EKS)
   ```text
   kubectl -n kube-system annotate storagecluster --all portworx.io/is-eks="true"
   ```
* OpenShift platform
   ```text
   kubectl -n kube-system annotate storagecluster --all portworx.io/is-openshift="true"
   ```

### Perform a dry run (optional)

{{<companyName>}} recommends running a dry run and reviewing the specs:

1. Find the Portworx Operator pod:

   ```text
   kubectl -n kube-system get pods -l name=portworx-operator
   ```

2. Execute the dry run tool in the Portworx Operator container:

   ```text
   kubectl -n kube-system exec -it <portworx-operator-pod> -- /dryrun
   ```

If the dry run is successful, the end of the output will say `Operator migration dry-run finished successfully`.

To review the specs that the Operator will deploy after migration, especially if you have any custom changes to your DaemonSet deployment, run the following commands:

1. List all of the specs that the Operator will deploy after migration:

   ```text
   kubectl -n kube-system exec -it <portworx-operator-pod> -- ls /tmp/portworxSpecs
   ```
2. Preview the Portworx pod spec after migration:

   ```text
   kubectl -n kube-system exec -it <portworx-operator-pod> -- cat /tmp/portworxSpecs/Pod.portworx.yaml
   ```

### Migration using OnDelete update strategy (optional)

If you are not using `OnDelete` for your StorageCluster update strategy, proceed to the [next step](#approve-the-migration).

If the StorageCluster update strategy is `OnDelete` (which means that DaemonSet has the `OnDelete` update strategy or the StorageCluster was updated manually to include this strategy), you will need to manually migrate Portworx pods on the nodes. Migration will be blocked until all Portworx nodes are migrated. After all Portworx nodes are migrated, Operator will migrate the components automatically.

1. Run the following command to approve the migration:
   
   ```text
   kubectl -n kube-system annotate storagecluster --all --overwrite portworx.io/migration-approved='true'
   ```
2. Change the migration label to `Starting` on the KVDB nodes:


   ```text
   kubectl label node <kvdb-node> portworx.io/daemonset-migration=Starting --overwrite
   ```
      A KVDB node should be migrated one at a time, and wait until all KVDB nodes are migrated and pods are created:

2. Change the migration label to `Starting` on nodes to trigger migration on a Portworx node:

   ```text
   kubectl label node <portworx-node> portworx.io/daemonset-migration=Starting --overwrite
   ```

   Operator starts migrating the labeled nodes at once.

{{<info>}}**NOTES**:

* If the update strategy is changed from `OnDelete` to `RollingUpdate` during the nodes migration, the Operator will pick up the Portworx node in progress and continue the migration process automatically.
* If the update strategy is changed from `RollingUpdate` to `OnDelete` during the nodes migration, the procedure will be blocked after the current node migration is done, then the procedure will wait for manual approval.
{{</info>}}

You can skip the next section and proceed to [this section](#check-migration-status) to verify the status of your migration.

### Approve the migration

To approve the migration, run the following command:

```tet
kubectl -n kube-system annotate storagecluster --all --overwrite portworx.io/migration-approved='true'
```

### Check migration status

Describe the StorageCluster using the following command:

```text
kubectl -n kube-system describe storagecluster
```

If the migration completes successfully, you will see the event `Migration completed successfully`. If migration fails, there is corresponding event about the failure.

## Rollback to DaemonSet

If the migration from DaemonSet to the Operator fails for some reason and you want to rollback to the DaemonSet installation, perform the following steps.

### Save DaemonSet backup to a file

Save the DaemonSet backup, which is automatically generated by the Operator, to a YAML file. Replace `kube-system` with the DaemonSet namespace in the following command:

```text
kubectl -n kube-system describe configmap px-daemonset-backup | grep -v Events | tail -n +10 > backup.yaml
```

### Remove Operator installation

To remove the Operator installation, perform the following steps:

1. Delete the Operator installation by deleting the StorageCluster CRD:

   ```text
   kubectl -n kube-system delete storagecluster --all
   ```

2. Delete Operator with the following command:

   ```text
   kubectl -n kube-system delete deployment portworx-operator
   ```

3. Remove the migration labels from the nodes:

   ```text
   kubectl label node --all --overwrite portworx.io/daemonset-migration-
   ```

### Apply the DaemonSet backup

Install with DaemonSet backup using the following command:

```text
kubectl apply -f backup.yaml
```

## Troubleshooting

### One or more Portworx nodes are already down for some known reason

If Portworx is down on a node, migration will get stuck on that node as it tries to ensure that Portworx is online before it can proceed to the next node. If you know that the Portworx node is expected to be down, you can skip that node and let the migration continue to the next node. Add the following label on the node on which Portworx is down:

```text
kubectl label node <node-name> portworx.io/daemonset-migration=Skip --overwrite
```

### Grafana isn’t working after migration

If the Grafana pod is running with the Prometheus endpoint `http://prometheus:9090`, then the Grafana portal won’t work after migrating to Operator from DaemonSet. Perform the following modifications:

1. Run the following command:

   ```text
   kubectl edit configmap grafana-source-config -n kube-system
   ```

2. Replace `url: http://prometheus:9090` with the following:

   ```text
   url: http://px-prometheus:9090
   ```

3. Bounce the Grafana pod by deleting the existing Grafana pod.

### Prometheus service name change

Prometheus should still work after migration, but the name of the service will change to `px-prometheus`, whereas the default service name in the DaemonSet installation is `prometheus`. To verify this, run the following command:

```text
kubectl get service -n kube-system 
```
```output
px-prometheus ClusterIP 10.233.54.146 <none> 9090/TCP
```
