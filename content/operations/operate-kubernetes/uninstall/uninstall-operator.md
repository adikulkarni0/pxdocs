---
title: Uninstall Portworx from a Kubernetes cluster using the Operator
linkTitle: Uninstall using the Operator
keywords: portworx, container, kubernetes, storage, k8s, uninstall, wipe, cleanup
description: Learn how to uninstall Portworx using the Operator.
weight: 200
aliases:
  - /portworx-install-with-kubernetes/on-premise/openshift/operator/uninstall/
  - /portworx-install-with-kubernetes/openshift/operator/uninstall
  - /portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/uninstall/uninstall-operator/
series: k8s-uninstall
---

If you're using the Portworx Operator, you can uninstall Portworx by adding a delete strategy to your `StorageCluster` object, then deleting it. When uninstalling, you may choose to either keep the the data on your drives, or wipe them completely:

* **Uninstall**: will remove the Kubernetes objects, Portworx `systemctl` service, `/etc/pwx` and `/opt/pwx` directories, and all traces of Portworx on the nodes. The drives will not be formatted and none the Portworx Metadata in the KVDB will be deleted. You may need to Uninstall Portworx if you installed Portworx in the wrong namespace. 
* **Uninstall and wipe**: will remove all of the resources listed in the "Uninstall" procedure, and also removes (formats) all data from your disks permanently, including the Portworx metadata. You may want to perform an uninstall and wipe when you decommission a cluster.


## Prerequisites

* You must already be running Portworx through the Operator, this method will not work for other Portworx deployments

## Uninstall Portworx

1. Enter the `kubectl get` command to display the name of your Portworx storage cluster and specify your namespace:

      ```text
      kubectl get -n kube-system storagecluster <storagecluster-name>
      ```

2. Enter the `kubectl edit` command to modify your storage cluster and specify your namespace:

      ```text
      kubectl edit -n kube-system storagecluster <storagecluster-name>
      ```

3. Modify your `StorageCluster` object, adding the `deleteStrategy` field with either the `Uninstall` or `UninstallAndWipe` type:

    * Uninstall Portworx only:

        ```text
        apiVersion: core.libopenstorage.org/v1
        kind: StorageCluster
        metadata:
          name: portworx
          namespace: kube-system
        spec:       
          deleteStrategy:
            type: Uninstall
        ```
    * Uninstall Portworx and wipe all drives:

        {{<info>}}
**WARNING:** Wipe operations remove all data from your disks permanently including the Portworx metadata, use caution when applying the DeleteStrategy spec.
        {{</info>}}

        ```text
        apiVersion: core.libopenstorage.org/v1
        kind: StorageCluster
        metadata:
          name: portworx
          namespace: kube-system
        spec:       
          deleteStrategy:
            type: UninstallAndWipe
        ```

4. Enter the `kubectl delete` command, specifying the name of your `StorageCluster` object and specify your namespace:

    ```text
    kubectl delete StorageCluster <your-storagecluster-name> -n kube-system
    ```
    {{<info>}}
**NOTE:** This operation can take several minutes to complete.
    {{</info>}}

5. Delete the operator:

  ```text
  kubectl delete deployment -n <namespace> portworx-operator
  ```