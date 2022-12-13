---
title: Encrypting PVCs using CSI and Kubernetes Secrets
linkTitle: PVC Encryption with CSI and Kubernetes
weight: 100
keywords: Portworx, Kubernetes, Kubernetes Secrets, containers, storage, encryption, CSI
description: Instructions on using Kubernetes Secrets with Portworx for encrypting PVCs on CSI using StorageClass
noicon: true
series: kubernetes-secret-uses
series2: k8s-pvc-enc
hidden: false
aliases:
    - /key-management/kubernetes-secrets/pvc-encryption-using-CSI/
---
This article discusses the PVC encryption methods used with the Kubernetes Container Storage Interface. For details about using Portworx with CSI, refer to the [Portworx with CSI](/operations/operate-kubernetes/storage-operations/csi/) page.

## Prerequisites

In order to perform the steps in this document, you must have [Portworx with CSI](/operations/operate-kubernetes/storage-operations/csi/) enabled.

## Encrypt your volumes

You can encrypt your volumes in one of two ways:

* Per storage class
* Per PVC

### Encrypt your volumes per storage class

You can encrypt your volumes by specifying the encryption key in a Kubernetes secret. This secret can be same as the one created to host the authentication token. Using this method, you can handle both authentication and encryption together, and multiple PVCs referring to this storage class can use the same secret for encryption.

#### Step 1: Create a Kubernetes secret that contains the passphrase used for encrypting the Portworx volume

Enter the following command, specifying your own passphrase in `mysecret-passcode-for-encryption`, which encrypts the PVC:

```text
kubectl create secret generic volume-secrets -n kube-system --from-literal=mysql-pvc-secret-key={{<highlight>}}<mysecret-passcode-for-encryption>{{</highlight>}}
```

#### Step 2: Create a CSI Kubernetes secret that points to the Kubernetes encryption secret

The CSI implementation reads the Kubernetes secret `px-secret` and passes its contents to Portworx. The `px-secret` must contain values for the following fields, which tell Portworx which Kubernetes secret to fetch the encryption passphrase from:

* `SECRET_NAME`: The name you have given your secret (in this case, `volume-secrets`).
* `SECRET_KEY`: The key for accessing the encryption key in the referenced `SECRET_CONTEXT`/`SECRET_NAME`.
* `SECRET_CONTEXT`: The namespace in which you created the secret (in this case, `kube-system`).

Enter the following command:

```text
kubectl create secret generic px-secret -n kube-system --from-literal=SECRET_NAME={{<highlight>}}volume-secrets{{</highlight>}} --from-literal=SECRET_KEY={{<highlight>}}mysql-pvc-secret-key{{</highlight>}} --from-literal=SECRET_CONTEXT={{<highlight>}}kube-system{{</highlight>}}
```

#### Step 3: Create a CSI storage class for encrypted PVC

Create the storage class which refers to the CSI secret you created in step 2 above. Specify the following:

  * `csi.storage.k8s.io/provisioner-secret-name`: the name of your CSI secret
  * `csi.storage.k8s.io/provisioner-secret-namespace`: the namespace in which your CSI secret is located
  * `csi.storage.k8s.io/node-publish-secret-name`: the name of your CSI secret
  * `csi.storage.k8s.io/node-publish-secret-namespace`: the namespace in which your CSI secret is located

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
      csi.storage.k8s.io/provisioner-secret-name: {{<highlight>}}px-secret{{</highlight>}}
      csi.storage.k8s.io/provisioner-secret-namespace: {{<highlight>}}kube-system{{</highlight>}}
      csi.storage.k8s.io/node-publish-secret-name: {{<highlight>}}px-secret{{</highlight>}}
      csi.storage.k8s.io/node-publish-secret-namespace: {{<highlight>}}kube-system{{</highlight>}}
    reclaimPolicy: Delete
    volumeBindingMode: Immediate
    allowVolumeExpansion: true
    ```


### Encrypt your volumes per PVC

You can encrypt volumes by allowing your users to specify encryption keys in their PVCs. Using this method, each PVC will use its own key for encryption. Follow the steps below to create two PVCs which use different passphrases for encryption:

#### Step 1: Create two Kubernetes secrets to house the encryption keys for your two PVCs

Kubernetes secrets can be in the Portworx namespace or in a user namespace. In this example we will be creating Kubernetes secrets in the user namespace `default`.

Enter the following command to create the secret for your first PVC, specifying your own passphrase for `mysecret-passcode-for-encryption`, which encrypts the PVC:

```text
kubectl create secret generic {{<highlight>}}volume-secrets-1{{</highlight>}} --from-literal=mysql-pvc-secret-key-1={{<highlight>}}mysecret-passcode-for-encryption-1{{</highlight>}}
```

Enter the same command to create the secret for your second PVC, optionally specifying a different secret name and a different namespace. This example places both PVCs in `default` namespace:

```text
kubectl create secret generic {{<highlight>}}volume-secrets-2{{</highlight>}} --from-literal=mysql-pvc-secret-key-2={{<highlight>}}mysecret-passcode-for-encryption-2{{</highlight>}}
```

#### Step 2: Create two additional Kubernetes secrets that refer to the encryption key secrets

Enter the following command to create another secret associated with your first PVC, specifying the following options:

  * This secret's name (`mysql-pvc-1` in this example)
  * The `-n` option with the same namespace which the first Kubernetes secret you created to house your encryption key points to (`csi-test-demo` in this example)
  * The `--from-literal=SECRET_NAME=` option and the name of the encryption key secret you created 
  * The `--from-literal=SECRET_KEY=` option and the key from inside the PVC's encryption key secret
  * The `--from-literal=SECRET_CONTEXT=` option and the encryption key secret's namespace as described in step 1

```text
kubectl create secret generic {{<highlight>}}mysql-pvc-1{{</highlight>}} -n {{<highlight>}}csi-test-demo{{</highlight>}} --from-literal=SECRET_NAME={{<highlight>}}volume-secrets-1{{</highlight>}} --from-literal=SECRET_KEY={{<highlight>}}mysql-pvc-secret-key-1{{</highlight>}} --from-literal=SECRET_CONTEXT={{<highlight>}}csi-test-demo{{</highlight>}}
```

Enter the same command for your second PVC, but specify a different secret name and optionally, a different namespace:

```text
kubectl create secret generic {{<highlight>}}mysql-pvc-2{{</highlight>}} -n {{<highlight>}}csi-test-demo{{</highlight>}} --from-literal=SECRET_NAME={{<highlight>}}volume-secrets-2{{</highlight>}} --from-literal=SECRET_KEY={{<highlight>}}mysql-pvc-secret-key-2{{</highlight>}} --from-literal=SECRET_CONTEXT={{<highlight>}}csi-test-demo{{</highlight>}}
```

#### Step 3: Create a CSI storage class for encrypted PVCs

Create a StorageClass CRD, specifying the `${pvc.name}` and `${pvc.namespace}` template variables:

```text
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: px-csi-db-encrypted-pvc-k8s
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

#### Step 4: Create encrypted PVCs

Create two encrypted PVCs, one for each of the secrets you created in the preceding steps:

1. Create the `mysql-pvc-1` PVC that uses the passcode you created [previously](/operations/key-management/kubernetes-secrets/pvc-encryption-using-csi/#step-1-create-two-kubernetes-secrets-to-house-the-encryption-keys-for-your-two-pvcs). In the example, that passcode is `mysecret-passcode-for-encryption-1`:

      ```text
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: {{<highlight>}}mysql-pvc-1{{</highlight>}}
        namespace: csi-test-demo
      spec:
        storageClassName: px-csi-db-encrypted-pvc-k8s
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 2Gi
      ```

2. Create the `mysql-pvc-2` PVC that uses the passcode you created [previously](/operations/key-management/kubernetes-secrets/pvc-encryption-using-csi/#step-1-create-two-kubernetes-secrets-to-house-the-encryption-keys-for-your-two-pvcs). In the example, that passcode is `mysecret-passcode-for-encryption-2`:

      ```text
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: {{<highlight>}}mysql-pvc-2{{</highlight>}}
        namespace: csi-test-demo
      spec:
        storageClassName: px-csi-db-encrypted-pvc-k8s
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 2Gi
      ```

The templatized parameters in the CSI storage class point to the name and namespace of the PVC itself. This ensures that each PVC requires a separate Kubernetes secret of the same name in the same namespace. In this way, each PVC gets encrypted with its own passphrase.
