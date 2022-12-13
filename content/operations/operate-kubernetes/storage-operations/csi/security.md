---
title: Secure your volumes 
keywords: csi, portworx, container, Kubernetes, storage, Docker, k8s, pv, security
description: This page explains how you can start securing your volume with PX-Security
weight: 400
aliases:
    - /portworx-install-with-kubernetes/storage-operations/csi/security/
---
# Encryption with CSI

For information about how to encrypt PVCs on CSI using Kubernetes secrets, see [encrypting PVCs on CSI with Kubernetes secrets](/operations/key-management/kubernetes-secrets/pvc-encryption-using-csi/).


# Authorization and Authentication

You can secure your CSI-enabled volumes with token-based authorization. In using token-based authorization, you create secrets containing your token credentials and specify them in your StorageClass in one of two ways:

* Using hardcoded values
* Using template values

You can also mix these two methods to form your own hybrid approach.

### Using hardcoded values

This example secures a storage class by specifying hardcoded values for the token and namespace. Users who create PVCs based on this StorageClass will always have their PVCs use the `px-user-token` StorageClass under the `kube-system` namespace.

1. Find or create your token secret:

    For operator deployment with security enabled, a user token is automatically created and refreshed under `px-user-token` in your `StorageCluster` namespace. Refer to [secure your storage with the Operator](/cloud-references/security/kubernetes/shared-secret-model-operator) for more information.

    ```text
    kubectl get secrets px-user-token -n kube-system
    ```
    ```output
    NAME            TYPE     DATA   AGE
    px-user-token   Opaque   1      23h
    ```

    For all other configurations, [create your own token secret](/cloud-references/security/kubernetes/shared-secret-model/generating-tokens/):
    
    ```text
    kubectl create secret generic px-user-token \
      -n kube-system --from-literal=auth-token=$USER_TOKEN
    ```

2. Modify the storageClass:

    * If you're using Kubernetes secrets, add the following parameters that are shown in the following example:

        ```text
        apiVersion: storage.k8s.io/v1
        kind: StorageClass
        metadata:
          name: px-csi-db-encrypted-k8s
        provisioner: pxd.portworx.com
        parameters:
          repl: "3"
          secure: "true"
          io_profile: auto
          io_priority: "high" 
          cow_ondemand: "false"
          {{<highlight>}}csi.storage.k8s.io/provisioner-secret-name: px-user-token
          csi.storage.k8s.io/provisioner-secret-namespace: kube-system
          csi.storage.k8s.io/node-publish-secret-name: px-user-token
          csi.storage.k8s.io/node-publish-secret-namespace: kube-system{{</highlight>}}
        reclaimPolicy: Delete
        volumeBindingMode: Immediate
        allowVolumeExpansion: true
        ```
      
    * If you're using another secret provider, such as Vault, Google KMS, AWS KMS, or KVDM, define the desired encryption key in the `secret_key` parameter directly as a parameter on the CSI storage class. For example:

        ```text
        apiVersion: storage.k8s.io/v1
        kind: StorageClass
        metadata:
          name: px-csi-db-encrypted-vault
        provisioner: pxd.portworx.com
        parameters:
          repl: "3"
          secure: "true"
          io_profile: auto
          io_priority: "high"
          cow_ondemand: "false"
          secret_key: {{<highlight>}}px-cluster-key{{</highlight>}}
        reclaimPolicy: Delete
        volumeBindingMode: Immediate
        allowVolumeExpansion: true
        ```
        
      {{<info>}}
**Note:** Ensure that the secret called `px-cluster-key` exists and contains the value that you can use to encrypt your volumes.
      {{</info>}}


### Using template values

This example secures a StorageClass by hardcoding the token and namespace. If you have created PVCs based on this StorageClass, you can have your PVCs use the StorageClass, which you specify in the annotation of your PVC, and the namespace specified in your PVC.

1. Modify the StorageClass, adding the following parameters:

    * `csi.storage.k8s.io/provisioner-secret-name: ${pvc.name}`
    * `csi.storage.k8s.io/provisioner-secret-namespace: ${pvc.namespace}`
    * `csi.storage.k8s.io/node-publish-secret-name: ${pvc.name}`
    * `csi.storage.k8s.io/node-publish-secret-namespace: ${pvc.namespace}`

       ```text
       apiVersion: storage.k8s.io/v1
       kind: StorageClass
       metadata:
          name: px-csi-db-encrypted-k8s
       provisioner: pxd.portworx.com
       parameters:
          repl: "3"
          secure: "true"
          io_profile: auto
          io_priority: "high" 
          cow_ondemand: "false"
          csi.storage.k8s.io/provisioner-secret-name: {{<highlight>}}${pvc.name}{{</highlight>}}
          csi.storage.k8s.io/provisioner-secret-namespace: {{<highlight>}}${pvc.namespace}{{</highlight>}}
          csi.storage.k8s.io/node-publish-secret-name: {{<highlight>}}${pvc.name}{{</highlight>}}
          csi.storage.k8s.io/node-publish-secret-namespace: {{<highlight>}}${pvc.namespace}{{</highlight>}}
       reclaimPolicy: Delete
       volumeBindingMode: Immediate
       allowVolumeExpansion: true
       ```
2. Create a secret in your PVC's namespace with the same name as the PVC. For example, a PVC named `px-mysql-pv`c"` must have an associated secret named `px-mysql-pvc`.


  1. Get the token as mentioned in [Using hardcoded values](#using-hardcoded-values):
    ```text
    USER_TOKEN=$(kubectl get secrets px-user-token -n kube-system -o json | jq -r '.data["auth-token"]' | base64 -d)
    ```
  2. Create the secret:
    ```text
    kubectl create secret generic px-mysql-pvc -n kube-system --from-literal=auth-token=$USER_TOKEN
    ```