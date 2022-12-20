---
title: "Step 3: StorageClass Setup"
keywords: storageclass, csi, security, authorization
hidden: true
---

In the previous section, you saved the Kubernetes token in a secret called
`px-user-token` in the `kube-system` namespace. Now you can create a StorageClass
which points Portworx to authenticate the request using the token in the
that secret.

Portworx validates requests to manage volumes using the token saved in the secret referenced by the StorageClass. As you create more 
StorageClasses, remember to reference the secret with the token to authenticate the requests. 
The example below demonstrates a StorageClass with token secrets added:

# StorageClass for CSI

When using [CSI](/operations/operate-kubernetes/storage-operations/csi/), the StorageClass references the secret for the three types
of supported operations: _provision_, _node-publish_ (mount/unmount), and
_controller-expand_.

1. Create the following `storageclass.yaml` file:

    ```
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: px-storage
    provisioner: pxd.portworx.com
    parameters:
      repl: "1"
      csi.storage.k8s.io/provisioner-secret-name: px-user-token
      csi.storage.k8s.io/provisioner-secret-namespace: kube-system
      csi.storage.k8s.io/node-publish-secret-name: px-user-token
      csi.storage.k8s.io/node-publish-secret-namespace: kube-system
      csi.storage.k8s.io/controller-expand-secret-name: px-user-token
      csi.storage.k8s.io/controller-expand-secret-namespace: kube-system
    allowVolumeExpansion: true
    ```

2. Apply the `storageclass.yaml` file:

    ```
    kubectl apply -f storageclass.yaml
    ```

# StorageClass for non-CSI

For StorageClasses using the (now [deprecated](https://kubernetes.io/blog/2022/09/26/storage-in-tree-to-csi-migration-status-update-1.25/#:~:text=Portworx,1.29%20(Target)) as of Kubernetes v1.25.x onward) in-tree Portworx driver, the approach to leverage PX-Security is as follows:

1. Create the following `storageclass.yaml` file:

    ```
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: px-storage
    provisioner: kubernetes.io/portworx-volume
    parameters:
      repl: "1"
      openstorage.io/auth-secret-name: px-user-token
      openstorage.io/auth-secret-namespace: kube-system
    allowVolumeExpansion: true
    ```

2. Apply the `storageclass.yaml` file:

    ```
    kubectl apply -f storageclass.yaml
    ```
