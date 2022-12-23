---
title: Generate and apply a cluster pair spec
linkTitle: Generate cluster pair
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration
description: How to pair your clusters.
series: setup-AysncDR
weight: 300
---

In Kubernetes, you must define a trust object called a **ClusterPair**. Portworx requires this object to communicate with the destination cluster. The ClusterPair object pairs the Portworx storage driver with the Kubernetes scheduler, allowing volumes and resources to be migrated between clusters.


{{<info>}}**IMPORTANT:** 

* In all examples, `<migrationnamespace>` is considered the admin namespace that will migrate all namespaces of your source cluster to the destination cluster. You can also specify a non-admin namespace, but only that namespace will be migrated. To learn how to set up an admin namespace, refer to the [Set up a Cluster Admin namespace for Migration](/operations/operate-kubernetes/migration/cluster-admin-namespace/) page.
* You must run the `pxctl` commands in this document either on your worker nodes or from the Portworx containers on your Kubernetes control plane node.    
{{</info>}} 


## Create object store credentials for cloud clusters

Create object store credentials on your source and destination clusters. In cloud-based Kubernetes clusters, the options for creating object store credentials differ depending on which object store you are using.

{{<info>}}
**IMPORTANT:** You must create object store credentials on both the destination and source clusters before you create a cluster pair.
{{</info>}}

### Create Amazon s3 or s3 compatible object store credentials

1. Find the UUID of your destination cluster by running the following command. Save this ID for use in the next step:
      ```text
      PX_POD=$(kubectl get pods -l name=portworx -n <namespace> -o jsonpath='{.items[0].metadata.name}')
      kubectl exec $PX_POD -n <namespace> --  /opt/pwx/bin/pxctl status | grep UUID | awk '{print $3}'
      ```

2. Create the credentials by running the `pxctl credentials create` command with the following flags, as shown in the example:

    * The `--provider` flag with the name of the cloud provider (`s3`).
    * The `--s3-access-key` flag with your secret access key.
    * The `--s3-secret-key` flag with your access key ID.
    * The `--s3-region` flag with the name of the s3 region (for example, `us-east-1`. Update this value to your object store region).
    * The `--s3-endpoint` flag with the name of the endpoint. For Amazon s3 object stores, use `s3.amazonaws.com`. For s3 compatible object stores, specify that object store's endpoint in the `https://<your-end-point.com>` format.
    * The optional `--s3-storage-class` flag with either the `STANDARD` or `STANDARD-IA` value, depending on which storage class you prefer.
    * `clusterPair_<UUID-of-destination-cluster>` with the UUID of your destination cluster retrieved in the previous step.


   ```text
   /opt/pwx/bin/pxctl credentials create \
   --provider s3 \
   --s3-access-key <secret-access-key> \
   --s3-secret-key <access-key-id> \
   --s3-region us-east-1  \
   --s3-endpoint s3.amazonaws.com \
   --s3-storage-class STANDARD \
   clusterPair_<UUID-of-destination-cluster>
   ```

### Create Microsoft Azure credentials

1. Find the UUID of your destination cluster by running the following command. Save this ID for use in the next step:
      ```text
      PX_POD=$(kubectl get pods -l name=portworx -n <namespace> -o jsonpath='{.items[0].metadata.name}')
      kubectl exec $PX_POD -n <namespace> --  /opt/pwx/bin/pxctl status | grep UUID | awk '{print $3}'
      ```

2.  Create the credentials by running the `pxctl credentials create` command with the following flags, as shown in the example:

    * `--provider` as `azure`.
    * `--azure-account-name` flag with the name of your Azure account.
    * `--azure-account-key` flag with your Azure account key.
    * `clusterPair_<UUID-of-destination-cluster>` with the UUID of your destination cluster retrieved in the previous step.

    ```text
    /opt/pwx/bin/pxctl credentials create \
    --provider azure \
    --azure-account-name <your-azure-account-name> \
    --azure-account-key <your-azure-account-key> \
    clusterPair_<UUID-of-destination-cluster>
    ```

### Create Google Cloud Platform credentials

1. Find the UUID of your destination cluster by running the following command. Save this ID for use in the next step:
      ```text
      PX_POD=$(kubectl get pods -l name=portworx -n <namespace> -o jsonpath='{.items[0].metadata.name}')
      kubectl exec $PX_POD -n <namespace> --  /opt/pwx/bin/pxctl status | grep UUID | awk '{print $3}'
      ```

2.  Create the credentials by running the `pxctl credentials create` command with the following flags, as shown in the example:

    * `--provider` as `google`
    * `--google-project-id` with the string of your Google project ID
    * `--google-json-key-file` with the filename of your GCP JSON key file
    * `clusterPair_<UUID-of-destination-cluster>` with the UUID of your destination cluster retrieved in the previous step.

    ```text
    /opt/pwx/bin/pxctl credentials create \
    --provider google \
    --google-project-id <your-google-project-ID> \
    --google-json-key-file <your-GCP-JSON-key-file> \
    clusterPair_<UUID-of-destination-cluster>
    ```


## Generate a ClusterPair spec on the destination cluster

Perform the following steps from your destination cluster:

1. Run the following command by specifying the `<remotecluster>` and `<migrationnamespace>`to create a ClusterPair and save the resulting spec to a file named `clusterpair.yaml`:

   * `<remotecluster>`: is the Kubernetes object that will be created on the **source** cluster representing the pair relationship.
   * `<migrationnamespace>`: is the Kubernetes namespace for the **source** cluster that you want to migrate to the **destination** cluster.

   ```text
   storkctl generate clusterpair -n <migrationnamespace> <remotecluster> -o yaml > clusterpair.yaml
   ```
 
 
 
2. Run the following command from one of the Portworx nodes to get the destination cluster token and save it for use in the next step:

   ```text
   PX_POD=$(kubectl get pods -l name=portworx -n <namespace> -o jsonpath='{.items[0].metadata.name}')
   kubectl exec $PX_POD -n <namespace> --  /opt/pwx/bin/pxctl cluster token show
   ```


3. Specify the following fields in the `options` section of the `clusterpair.yaml` file that you saved previously to pair the storage:

   * `ip`: Specify the IP address of any remote Portworx nodes. If using an external load balancer, specify the IP address of the external load balancer.
   * `port`: Specify the port of the remote Portworx node mentioned in the Network Connectivity section on the Prerequisites page.
   * `token`: Specify the token of the destination cluster obtained from the previous step.
   * `mode`: Specify `DisasterRecovery`. By default, every seventh migration is a full migration. If you specify `mode: DisasterRecovery`, then every migration is incremental. When doing a one-time migration (and not DR), skip this option.


4. (Optional) If you are using PX-Security on both source and destination clusters, you will need to add the following two annotations to the `ClusterPair` and `MigrationSchedule` specs:
   ```text
   annotations:
      openstorage.io/auth-secret-namespace -> Points to the namespace where the k8s secret holding the auth token resides.
      openstorage.io/auth-secret-name -> Points to the name of the k8s secret which holds the auth token.
   ```
   
  
    {{<info>}}**NOTE:** The values of all parameters in the `options` section of the updated specs should be in double quotes, as shown in the following example:

```text
options:
   token: "XXXX"
   ip: "192.0.2.0"
   port: "9001"
   mode: "DisasterRecovery"
   ```
   {{</info>}}   


## Apply the ClusterPair spec on the source cluster

Perform the following steps from your source cluster:

1. Apply the updated ClusterPair spec on your source cluster. Run the following command from a location where you have `kubectl` access to your source cluster:

   ```text
   kubectl apply -f clusterpair.yaml -n <migrationnamespace>
   ```


2. Verify that the ClusterPair spec is generated, and it is ready:

   ```text
   storkctl get clusterpair -n <migrationnamespace>
   ```
   ```output
   NAME            STORAGE-STATUS   SCHEDULER-STATUS   CREATED
   remotecluster   Ready            Ready              09 Nov 22 00:22 UTC
   ```

      On a successful pairing, you should see the `STORAGE-STATUS` and `SCHEDULER-STATUS` as `Ready`.

If you see an error, run the following command to get more information: 

```text
kubectl describe clusterpair remotecluster -n <migrationnamespace>
```

