---
title: Shared
hidden: true
keywords: portworx, kubernetes, DR
---
## Generate and Apply a ClusterPair Spec

In Kubernetes, you must define a trust object called **ClusterPair**. Portworx requires this object to communicate with the destination cluster. The ClusterPair object pairs the Portworx storage driver with the Kubernetes scheduler, allowing the volumes and resources to be migrated between clusters.

The ClusterPair is generated from and used in the following way:

   * It requires `storkctl` to be running on Kubernetes control plane node in both **destination** and **source** cluster
   * The **ClusterPair** spec is generated on the **destination** cluster 
   * The generated spec is then applied on the **source** cluster

Perform the following steps to create a cluster pair:

{{<info>}}
**NOTE:** You must run the `pxctl` commands in this document either on your worker nodes directly, or from inside the Portworx containers on your Kubernetes control plane node. 
{{</info>}} 

### Create object store credentials for cloud clusters

**You must create object store credentials on both the destination and source clusters before you can create a cluster pair.**
The options you use to create your object store credentials differ based on which object store you use:

#### Create Amazon s3 credentials

1. Find the UUID of your destination cluster

2. Enter the `pxctl credentials create` command, specifying the following:

    * The `--provider` flag with the name of the cloud provider (`s3`).
    * The `--s3-access-key` flag with your secret access key
    * The `--s3-secret-key` flag with your access key ID
    * The `--s3-region` flag with the name of the S3 region (`us-east-1`)
    * The `--s3-endpoint` flag with the  name of the endpoint (`s3.amazonaws.com`)
    * The optional `--s3-storage-class` flag with either the `STANDARD` or `STANDARD-IA` value, depending on which storage class you prefer
    * `clusterPair_<UUID_of_destination_cluster>` Enter the following command into your cluster to find its UUID:
      ```text
      PX_POD=$(kubectl get pods -l name=portworx -n kube-system -o jsonpath='{.items[0].metadata.name}')
      kubectl exec $PX_POD -n kube-system --  /opt/pwx/bin/pxctl status | grep UUID | awk '{print $3}'
      ```

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

1. Find the UUID of your destination cluster

2. Enter the `pxctl credentials create` command, specifying the following:

    * `--provider` as `azure`
    * `--azure-account-name` with the name of your Azure account
    * `--azure-account-key` with your Azure account key
    * `clusterPair_<UUID_of_destination_cluster>` Enter the following command into your cluster to find its UUID:
      ```text
      PX_POD=$(kubectl get pods -l name=portworx -n kube-system -o jsonpath='{.items[0].metadata.name}')
      kubectl exec $PX_POD -n kube-system --  /opt/pwx/bin/pxctl status | grep UUID | awk '{print $3}'
      ```

    ```text
    /opt/pwx/bin/pxctl credentials create \
    --provider azure \
    --azure-account-name <your_azure_account_name> \
    --azure-account-key <your_azure_account_key> \
    clusterPair_<UUID_of_destination_cluster>
    ```

#### Create Google Cloud Platform credentials

1. Find the UUID of your destination cluster

2. Enter the `pxctl credentials create` command, specifying the following:

    * `--provider` as `google`
    * `--google-project-id` with the string of your Google project ID
    * `--google-json-key-file` with the filename of your GCP JSON key file
    * `clusterPair_<UUID_of_destination_cluster>` Enter the following command into your cluster to find its UUID:
      ```text
      PX_POD=$(kubectl get pods -l name=portworx -n kube-system -o jsonpath='{.items[0].metadata.name}')
      kubectl exec $PX_POD -n kube-system --  /opt/pwx/bin/pxctl status | grep UUID | awk '{print $3}'
      ```

    ```text
    /opt/pwx/bin/pxctl credentials create \
    --provider google \
    --google-project-id <your_google_project_ID> \
    --google-json-key-file <your_GCP_JSON_key_file> \
    clusterPair_<UUID_of_destination_cluster>
    ```

### Generate a ClusterPair on the destination cluster

To generate the ClusterPair spec, run the following command on the **destination** cluster:

```text
storkctl generate clusterpair -n <migrationnamespace> <remotecluster>
```
Here, `remotecluster` is the Kubernetes object that will be created on the **source** cluster representing the pair relationship, and `migrationnamespace` is the Kubernetes namespace of the **source** cluster that you want to migrate to the **destination** cluster.

During the actual migration, you will reference this name to identify the destination of your migration.

{{<info>}}
**NOTE:** If you plan to enable PX-Security on both source and destination clusters, you will need to add the following two annotations to the `ClusterPair` and `MigrationSchedule`.
{{</info>}}


```text
  annotations:
    openstorage.io/auth-secret-namespace -> Points to the namespace where the k8s secret holding the auth token resides.
    openstorage.io/auth-secret-name -> Points to the name of the k8s secret which holds the auth token.
```


```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: ClusterPair
metadata:
    creationTimestamp: null
    name: remotecluster
    namespace: migrationnamespace
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
       mode: DisasterRecovery
status:
  remoteStorageId: ""
  schedulerStatus: ""
  storageStatus: ""
```

Save the resulting spec to a file named `clusterpair.yaml`.

Example: 
```text
storkctl generate clusterpair -n <migrationnamespace> <remotecluster> -o yaml > clusterpair.yaml
```

### Get the destination cluster token

On the destination cluster, run the following command from one of the Portworx nodes to get the cluster token. You'll need this token in later steps:

```text
pxctl cluster token show
```

### Insert Storage Options

{{<info>}}**NOTE:** When using ClusterPair in the context of Async DR, you must have a DR enabled Portworx license at both the source _and_ destination cluster to enable disaster recovery mode. If you do not have these licenses, enabling disaster recovery mode will fail.{{</info>}}

To pair the storage, specify the following fields in the `options` section of the `clusterpair.yaml` file saved previously:

* `ip`, with the IP address of any of the remote Portworx nodes
* `port`, with the port of the remote Portworx node
* `token`, the token of the destination cluster obtained fom the previous step.
* `mode`: by default, every seventh migration is a full migration. If you specify `mode: DisasterRecovery`, then every migration is incremental. When doing a one time Migration (and not DR) this option can be skipped.

{{<info>}}
In the updated spec, ensure values for all fields under options are quoted.
{{</info>}}

See example below:
```text
  options:
    token: "34b2fd8df839650ed8f9b5cd5a70e1ca6d79c1ebadb5f50e29759525a20aee7daa5aae35e1e23729a4d9f673ce465dbe3679032d1be7a1bcc489049a3c23a460"
    ip: "10.13.21.125"
    port: "9001"
    mode: "DisasterRecovery"
```

### Using Rancher Projects with ClusterPair

{{<info>}}**NOTE**: If you are not using Rancher, skip to the [next section](#apply-the-clusterpair-spec-on-the-source-cluster). {{</info>}}

Rancher has a concept of Projects that allow grouping of resources and Kubernetes namespaces. Depending on the resource and how it is created, Rancher adds the following label or annotation:
```text
field.cattle.io/projectID: <project-short-UUID>
```
The `projectID` uniquely identifies the project, and the annotation or label on the Kubernetes object provides a way to tie a Kubernetes object back to a Rancher project. 

From version 2.11.2 or newer, Stork has the capability to map projects from the source cluster to the destination cluster when it migrates Kubernetes resources. It will ensure that the following are transformed
when migrating Kubernetes resources to a destination cluster:
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
The project mappings are provided as a comma-separate key=value pairs. In this example, `projectID-A1` on source cluster maps to `projectID-A2` on the destination cluster, while `projectID-B1` on the source cluster maps to `projectID-B2`
on the destination cluster.


### Apply the ClusterPair spec on the source cluster

A typical ClusterPair spec should look like the following example once you have followed the previous steps

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: ClusterPair
metadata:
  creationTimestamp: null
  name: remotecluster
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
    ip:     <ip_of_any_remote_px_node>
    port:   <port_of_remote_px_node_default_9001>
    token:  <token_from_step_with_pxctl cluster token show>
    mode: DisasterRecovery
  platformOptions:
    # This section will be set only when using Rancher
    rancher:
      projectMappings:
        <projectID-A1>: <projectID-A2>
        <projectID-B1>: <projectID-B2>

status:
  remoteStorageId: ""
  schedulerStatus: ""
  storageStatus: ""
```

To create the ClusterPair, apply the ClusterPair YAML spec on the source cluster. Run the following command from a location where you have `kubectl` access to the source cluster:

```text
kubectl apply -f clusterpair.yaml -n <namespace>
```

### Verify the ClusterPair

To verify that you have generated the ClusterPair and that it is ready, run the following command:

```text
storkctl -n <namespace> get clusterpair
```
```output
NAME            STORAGE-STATUS   SCHEDULER-STATUS   CREATED
remotecluster   Ready            Ready              09 Nov 22 00:22 UTC
```

On a successful pairing, you should see the `STORAGE-STATUS` and `SCHEDULER-STATUS` as `Ready`.

If you see an error instead, you can get more information by running the following command:
```text
kubectl describe clusterpair remotecluster -n <namespace>
```

{{<info>}}
**NOTE:** You might need to perform additional steps if you are using [GKE](/operations/operate-kubernetes/migration/gke) or [EKS](/operations/operate-kubernetes/migration/eks).
{{</info>}}