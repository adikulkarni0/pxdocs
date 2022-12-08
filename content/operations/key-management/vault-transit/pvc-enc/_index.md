---
title: Encrypting Kubernetes PVCs with Vault Transit
weight: 100
keywords: Vault Transit `Key Management, Hashicorp, encrypt PVC, Kubernetes, k8s, Vault Namespaces
description: Instructions on using Vault Transit with Portworx for encrypting PVCs in Kubernetes
noicon: true
series: vault-transit-secret-uses
hidden: true
aliases:
    - /key-management/vault-transit/pvc-enc/
---
{{< content "shared/key-management-intro.md" >}}

### Encryption using StorageClass

In this method, each volume will use its own unique passphrase for encryption. Portworx relies on vault transit secrets engine to generate a Data Encryption Key. This key will then be used to encrypt and decrypt your volumes.


#### Step 1: Create a StorageClass

{{< content "shared/key-management-enc-storage-class-spec.md" >}}


#### Step 2: Create a PVC

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: secure-mysql-pvc
spec:
  storageClassName: px-secure-sc
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```


If you do not want to specify the `secure` flag in the StorageClass, but you want to encrypt the PVC using that StorageClass, then create the PVC as below:

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: secure-pvc
  annotations:
    px/secure: "true"
spec:
  storageClassName: portworx-sc
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```


### Encryption using PVC annotations with Vault Namespaces

If you have **Vault Namespaces** enabled and your secret resides inside a specific namespace, you must provide the name of that namespace and the secret key to Portworx.

#### Step 1: Create a StorageClass

{{< content "shared/key-management-enc-storage-class-spec.md" >}}

#### Step 2: Create a PVC with annotations

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: secure-mysql-pvc
  annotations:
    px/vault-namespace: <your-vault-namesapce>
spec:
  storageClassName: px-secure-sc
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

The PVC requires an extra annotation `px/vault-namespace` to indicate the Vault namespace where the secret key resides. If your key resides in the global vault namespace
set in Portworx using the parameter `VAULT_NAMESPACE`, you don't need to specify this annotation. However if the key resides in any other namespace then this annotation is
required.


### Encryption using PVC annotations with cluster wide secrets

#### Step 1: Create cluster wide secret

{{< content "shared/key-management-set-cluster-wide-secret.md" >}}


#### Step 2: Create a StorageClass

{{< content "shared/key-management-enc-storage-class-spec.md" >}}

#### Step 3: Create a PVC with annotations

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: secure-mysql-pvc
  annotations:
    px/secret-name: default
spec:
  storageClassName: px-secure-sc
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```


{{<info>}}
**IMPORTANT:** Portworx only allows default key for `px/secret-name` annotation for cluster wide secrets
{{</info>}}
