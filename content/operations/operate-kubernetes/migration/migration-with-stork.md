---
title: "Migration with Stork on Kubernetes"
linkTitle: "Migration with Stork"
keywords: cloud, backup, restore, snapshot, DR, migration, migration
description: How to migrate stateful applications on Kubernetes
series: migrate-with-stork
noicon: true
weight: 100
aliases:
  - /cloud-references/migration/migration-stork.html
  - /cloud-references/migration/migration-stork
  - /portworx-install-with-kubernetes/migration/
---

This document will walk you through how to migrate your Portworx volumes between clusters with Stork on Kubernetes.


## Prerequisites

Before we begin, please make sure the following prerequisites are met:

* **Version**: To take advantage of the features described in this document, the source AND destination clusters must have Stork 2.12.0 or later, Operator 1.10.0 or later, and {{< pxEnterprise >}} 2.11.0 or later.
* **Stork helper** : `storkctl` is a command-line tool for interacting with a set of scheduler extensions.
* **Secret Store** : Make sure you have configured a [secret store](/operations/key-management) on both clusters. This will be used to store the credentials for the objectstore.
* **Network Connectivity**:
  * Portworx: Port 9001 on the destination cluster should be reachable by the source cluster. For OpenShift, Port 17001 should be reachable.
  * Kubernetes API End Point: Port 6443, 443, or 8443 should be reachable. Check with your Kubernetes administrator to determine which port you need.

{{<info>}}
**NOTE:** Parts of this document require you to specify a `<namespace>`. Specify the namespace where Portworx is installed. Typically, Portworx is installed in either the `kube-system` or `portworx` namespace.
{{</info>}}

## Download storkctl

{{< content "shared/portworx-install-with-kubernetes-disaster-recovery-stork-helper.md" >}}

In Kubernetes, you must define a trust object called **ClusterPair**. Portworx requires this object to communicate with the destination cluster. The ClusterPair object pairs the Portworx storage driver with the Kubernetes scheduler, allowing volumes and resources to be migrated between clusters.

The ClusterPair is generated and used in the following way:

   * It requires `storkctl` to be running on the Kubernetes control plane node in both the **destination** and **source** clusters
   * The **ClusterPair** spec is generated on the **destination** cluster 
   * The generated spec is then applied on the **source** cluster

Perform the following steps to create a cluster pair:

{{<info>}}
**NOTE:** You must run the `pxctl` commands in this document either on your worker nodes directly, or from inside the Portworx containers on your Kubernetes control plane nodes. 
{{</info>}} 

## Create object store credentials for cloud clusters

Create object store credentials on your source and destination clusters. The options you use to create your object store credentials differ based on which object store you use.

{{<info>}}
**IMPORTANT:** You must create object store credentials on both the destination and source clusters before you can create a cluster pair.
{{</info>}}

#### Create Amazon s3 or s3 compatible object store credentials

1. Find the UUID of your destination cluster by running the following command:
      ```text
      PX_POD=$(kubectl get pods -l name=portworx -n <namespace> -o jsonpath='{.items[0].metadata.name}')
      kubectl exec $PX_POD -n <namespace> --  /opt/pwx/bin/pxctl status | grep UUID | awk '{print $3}'
      ```

2. Enter the `pxctl credentials create` command, specifying the following:

    * The `--provider` flag with the name of the cloud provider (`s3`).
    * The `--s3-access-key` flag with your secret access key
    * The `--s3-secret-key` flag with your access key ID
    * The `--s3-region` flag with the name of the S3 region (`us-east-1`)
    * The `--s3-endpoint` flag with the  name of the endpoint (`s3.amazonaws.com`)
    * The optional `--s3-storage-class` flag with either the `STANDARD` or `STANDARD-IA` value, depending on which storage class you prefer
    * `clusterPair_<UUID_of_destination_cluster>` Enter the UUID of your destination cluster collected in step 1. 

    ```text
    /opt/pwx/bin/pxctl credentials create \
    --provider s3 \
    --s3-access-key <aws_access_key> \
    --s3-secret-key <aws_secret_key> \
    --s3-region us-east-1  \
    --s3-endpoint s3.amazonaws.com \
    --s3-storage-class STANDARD \
    clusterPair_<UUID_of_destination_cluster>
    ```

#### Create Microsoft Azure credentials

1. Find the UUID of your destination cluster by running the following command:
      ```text
      PX_POD=$(kubectl get pods -l name=portworx -n <namespace> -o jsonpath='{.items[0].metadata.name}')
      kubectl exec $PX_POD -n <namespace> --  /opt/pwx/bin/pxctl status | grep UUID | awk '{print $3}'
      ```

2. Enter the `pxctl credentials create` command, specifying the following:

    * `--provider` as `azure`
    * `--azure-account-name` with the name of your Azure account
    * `--azure-account-key` with your Azure account key
    * `clusterPair_<UUID_of_destination_cluster>` Enter the UUID of your destination cluster collected in step 1.

    ```text
    /opt/pwx/bin/pxctl credentials create \
    --provider azure \
    --azure-account-name <your_azure_account_name> \
    --azure-account-key <your_azure_account_key> \
    clusterPair_<UUID_of_destination_cluster>
    ```

#### Create Google Cloud Platform credentials

1. Find the UUID of your destination cluster by running the following command:
      ```text
      PX_POD=$(kubectl get pods -l name=portworx -n <namespace> -o jsonpath='{.items[0].metadata.name}')
      kubectl exec $PX_POD -n <namespace> --  /opt/pwx/bin/pxctl status | grep UUID | awk '{print $3}'
      ```

2. Enter the `pxctl credentials create` command, specifying the following:

    * `--provider` as `google`
    * `--google-project-id` with the string of your Google project ID
    * `--google-json-key-file` with the filename of your GCP JSON key file
    * `clusterPair_<UUID_of_destination_cluster>` Enter the UUID of your destination cluster collected in step 1.

    ```text
    /opt/pwx/bin/pxctl credentials create \
    --provider google \
    --google-project-id <your_google_project_ID> \
    --google-json-key-file <your_GCP_JSON_key_file> \
    clusterPair_<UUID_of_destination_cluster>
    ```

### Using Rancher Projects with ClusterPair

{{<info>}}**NOTE**: If you are not using Rancher, skip to the [next section](#generate-a-clusterpair-from-the-destination-cluster). {{</info>}}

Rancher has a concept of Projects that allow grouping of resources and Kubernetes namespaces. Depending on the resource and how it is created, Rancher adds the following label or annotation:
```text
field.cattle.io/projectID: <project-short-UUID>
```
The `projectID` uniquely identifies the project, and the annotation or label on the Kubernetes object provides a way to tie a Kubernetes object back to a Rancher project. 

From version 2.11.2 or newer, Stork has the capability to map projects from the source cluster to the destination cluster when it migrates Kubernetes resources. It will ensure that the following are transformed when migrating Kubernetes resources to a destination cluster:

* Labels and annotations for projectID `field.cattle.io/projectID` on any Kubernetes resource on the source cluster are transformed to their respective projectIDs on the destination cluster.
* Namespace Selectors on a NetworkPolicy object which refer to the `field.cattle.io/projectID` label will be transformed to their respective projectIDs on the destination cluster.
* Namespace Selectors on a Pod object (Kubernetes version 1.24 or newer) which refer to the `field.cattle.io/projectID` label will be transformed to their respective projectIDs on the destination cluster.

{{<info>}}
**NOTE:**

* Rancher project mappings are supported only with Stork version 2.11.2 or newer.
* All the Rancher projects need to be created on both the source and the destination cluster.
{{</info>}}

While creating the ClusterPair, use the argument `--project-mappings` to indicate which projectID on the source cluster maps to a projectID on the destination cluster. 

For example:

```text
storkctl generate clusterpair -n <migrationnamespace> <remotecluster> --project-mappings  <projectID-A1>=<projectID-A2>,<projectID-B1>: <projectID-B2>
```
The project mappings are provided as a comma-separate `key=value` pairs. In this example, `projectID-A1` on source cluster maps to `projectID-A2` on the destination cluster, while `projectID-B1` on the source cluster maps to `projectID-B2`
on the destination cluster.

### Generate a ClusterPair from the destination cluster

{{<info>}}
**IMPORTANT**: The `<migrationnamespace>` in this example is an admin namespace. The `<migrationnamespace>` can also be a non-admin namespace. By default, `kube-system` is the default admin namespace, but another namespace can also be defined as an admin namespace. An admin namespace is a special namespace because any user with access to that namespace will be able to migrate any other namespace. If you only have access to a non-admin namespace, you can migrate only that namespace. For more information on setting up an admin namespace, see [cluster-admin-namespace](/operations/operate-kubernetes/migration/cluster-admin-namespace/).
{{</info>}}

To generate the ClusterPair spec, run the following command on the **destination** cluster:

```text
storkctl generate clusterpair -n <migrationnamespace> <remotecluster>
```

Here, `remotecluster` is the Kubernetes object that will be created on the **source** cluster representing the pair relationship, and `migrationnamespace` is the Kubernetes namespace of the **source** cluster that you want to migrate to the **destination** cluster.

During the actual migration, you will reference this name to identify the destination of your migration.

{{<info>}}
**NOTE:** If you are using PX-Security on both the source and destination clusters, you will need to add the following two annotations to the `ClusterPair` and `MigrationSchedule`:

```text
  annotations:
    openstorage.io/auth-secret-namespace -> Points to the namespace where the k8s secret holding the auth token resides.
    openstorage.io/auth-secret-name -> Points to the name of the k8s secret which holds the auth token.
```
{{</info>}}

Save the resulting spec to a file named `clusterpair.yaml`.

Example: 
```text
storkctl generate clusterpair -n <migrationnamespace> <remotecluster> -o yaml > clusterpair.yaml
```

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: ClusterPair
metadata:
    creationTimestamp: null
    name: <remotecluster>
    namespace: <migrationnamespace>
    annotations:
      # Add the below annotations when PX-Security is enabled on both the clusters
      #openstorage.io/auth-secret-namespace -> Points to the namespace where the k8s secret holding the auth token resides
      #openstorage.io/auth-secret-name -> Points to the name of the k8s secret which holds the auth token.
spec:
   config:
      clusters:
         kubernetes:
            LocationOfOrigin: /etc/kubernetes/admin.conf
            certificate-authority-data: <CA_DATA>
            server: https://192.168.56.74:6443
      contexts:
         kubernetes-admin@kubernetes:
            LocationOfOrigin: /etc/kubernetes/admin.conf
            cluster: kubernetes
            user: kubernetes-admin
      current-context: kubernetes-admin@kubernetes
      preferences: {}
      users:
         kubernetes-admin:
            LocationOfOrigin: /etc/kubernetes/admin.conf
            client-certificate-data: <CLIENT_CERT_DATA>
            client-key-data: <CLIENT_KEY_DATA>
    options:
       <insert_storage_options_here>: ""
status:
  remoteStorageId: ""
  schedulerStatus: ""
  storageStatus: ""
```

### Get the destination cluster token

On the destination cluster, run the following command from one of the Portworx nodes to get the cluster token. You'll need this token in later steps:

```text
PX_POD=$(kubectl get pods -l name=portworx -n <namespace> -o jsonpath='{.items[0].metadata.name}')
kubectl exec $PX_POD -n <namespace> --  /opt/pwx/bin/pxctl cluster token show
```

### Insert Storage Options

To pair the storage, specify the following fields in the `options` section of the `clusterpair.yaml` file saved previously:

* `ip`: The IP address of any of the remote Portworx nodes. If using external load balancer, specify the IP address of the external load balancer.
* `port`: The port of the remote Portworx node. See Network Connectivity in the [Prerequisites](#prerequisites) section of this document.
* `token`: The token of the destination cluster, which you obtained in the previous step.

See example below:
```text
  options:
    token: "34b2fd8df839650ed8f9b5cd5a70e1ca6d79c1ebadb5f50e29759525a20aee7daa5aae35e1e23729a4d9f673ce465dbe3679032d1be7a1bcc489049a3c23a460"
    ip: "10.13.21.125"
    port: "9001"
```

### Apply the ClusterPair spec on the source cluster

To create the ClusterPair, apply the ClusterPair YAML spec on the source cluster. Run the following command from a location where you have `kubectl` access to the source cluster:

```text
kubectl create -f clusterpair.yaml
```

### Verify the ClusterPair

To verify that you have generated the ClusterPair and that it is ready, run the following command:

```text
storkctl get clusterpair -n <migrationnamespace>
```
```output
NAME            STORAGE-STATUS   SCHEDULER-STATUS   CREATED
remotecluster   Ready            Ready              09 Nov 22 00:22 UTC
```

On a successful pairing, you should see the `STORAGE-STATUS` and `SCHEDULER-STATUS` as `Ready`.

If you see an error instead, you can get more information by running the following command:
```text
kubectl describe clusterpair remotecluster -n <migrationnamespace>
```

{{<info>}}
**NOTE:** You might need to perform additional steps if you are using [GKE](/operations/operate-kubernetes/migration/gke) or [EKS](/operations/operate-kubernetes/migration/eks).
{{</info>}}


## Migrating volumes and resources

Once the pairing is configured, applications can be migrated repeatedly to the destination cluster.

{{<info>}}
**NOTE:** If your cluster has a DR license applied to it, you can perform migrations only in DR mode; this includes operations involving the `pxctl cluster migrate` command.
{{</info>}}


## Start a migration

1. Create a file called `migration.yaml` file, specifying the following fields and values:

  * **apiVersion:** as `stork.libopenstorage.org/v1alpha1`
  * **kind:** as `Migration`
  * **metadata.name:** with the name of the object that performs the migration
  * **metadata.namespace:** with the name of the namespace in which you want to create the object
  * **spec.clusterPair:** with the name of the `ClusterPair` object created in the [Generate a ClusterPair from the destination cluster](#generate-a-clusterpair-from-the-destination-cluster) section
  * **spec.includeResources:** with a boolean value specifying if the migration should include PVCs and other applications specs. If you set this field to `false`, Portworx will only migrate your volumes.
  * **spec.startApplications:** with a boolean value specifying if Portworx should automatically start applications on the destination cluster. If you set this field to `false`, then the `Deployment` and `StatefulSet` objects on the destination cluster will be set to zero. Note that, on the destination cluster, Portworx uses the `stork.openstorage.org/migrationReplicas` annotation to store the number of replicas from the source cluster.
  * **spec.namespaces**: with the list of namespaces you want to migrate
  * **spec.adminClusterPair:** with the name of the `ClusterPair` object created in the admin namespace that Portworx should use to migrate your cluster-scoped resources. Use this if your regular users lack permission to create or delete cluster-scoped resources on the destination cluster. If you don't specify this field, then Stork will use the object specified in the `spec.clusterPair` field. This feature was introduced in Stork version 2.3.0.
  * **spec.purgeDeletedResources:** with a boolean value specifying if Stork should automatically purge a resource from the destination cluster when you delete it from the source cluster. The default value is `false`. This feature was introduced in Stork version 2.3.2.

    ```text
    apiVersion: stork.libopenstorage.org/v1alpha1
    kind: Migration
    metadata:
      name: <YOUR_MIGRATION_OBJECT>
      namespace: <YOUR_MIGRATION_NAMESPACE>
    spec:
      clusterPair: <YOUR_CLUSTER_PAIR>
      includeResources: true
      startApplications: true
      namespaces:
      - <NAMESPACE_TO_MIGRATE>
      adminClusterPair: <YOUR_ADMIN_CLUSTER_PAIR>
      purgeDeletedResources: false
    ```

2. Apply the spec by entering the following command:

    ```text
    kubectl apply -f migration.yaml
    ```

{{<info>}}
**Stork users:** You can run the following example command to create a migration:

```text
storkctl create migration <YOUR_MIGRATION_OBJECT> --clusterPair <YOUR_CLUSTER_PAIR> --namespaces <NAMESPACE_TO_MIGRATE> --includeResources --startApplications -n <YOUR_NAMESPACE>
```
{{</info>}}

#### Migration scope

Currently, you can only migrate namespaces in which the object is created. You can also designate one namespace as an admin namespace. This will allow an admin who has access to that namespace to migrate any namespace from the source cluster. Instructions for setting this admin namespace to `stork` can be found in [Set up a cluster admin namespace](/operations/operate-kubernetes/migration/cluster-admin-namespace/).

### Monitoring a migration

Once the migration has been started using the previous commands, you can check the status using `storkctl`. For example:

```text
storkctl get migration -n <migrationnamespace>
```

First, you should see something like the following:

```
NAME                                             CLUSTERPAIR     STAGE     STATUS       VOLUMES   RESOURCES   CREATED
zoomigrationschedule-interval-2022-12-12-200210  remotecluster   Volumes   InProgress   0/3       0/0         12 Dec 22 11:45 PST
```

If the migration is successful, the `STAGE` will go from `Volumes` to `Application` to `Final`.

Here is how the example output of a successful migration looks:

```output
NAME                                              CLUSTERPAIR   STAGE   STATUS       VOLUMES   RESOURCES   CREATED               ELAPSED
zoomigrationschedule-interval-2022-12-12-200210   remotecluster Final   Successful   3/3       10/10       12 Dec 22 12:02 PST   1m23s
```

{{< content "shared/portworx-install-with-kubernetes-disaster-recovery-migration-common.md" >}}
