---
title: Kubernetes Secrets
logo: /logos/other.png
keywords: Kubernetes Secrets, k8s, encryption keys, secrets, Volume Encryption, Cloud Credentials, secret store
description: Instructions on using Kubernetes secrets with Portworx
weight: 400
disableprevnext: true
series: key-management
noicon: true
aliases:
    - /key-management/kubernetes-secrets/
---
Portworx can integrate with Kubernetes Secrets to store your encryption keys/secrets and credentials. These encryption keys or secrets also support encrypting data at rest. Moreover, Portworx can utilize Kubernetes Secrets to store credentials and encryption keys associated with your cloud provider services.

The instructions on this page guides you to configure Portworx with Kubernetes Secrets. 



## Set Kubernetes Secrets as the secrets store

While installing Portworx on Kubernetes using the StorageCluster spec via [PX-Central](https://central.portworx.com), select **Kubernetes** from the **Secrets Store Type** list under **Advanced Settings**. To know how to generate Portworx spec, see the [Portworx Install](/install-portworx/) section.



## Create secrets with Kubernetes

The following section describes the key generation process with Portworx and Kubernetes which can be used for encrypting volumes.

### Set cluster wide secret key

A cluster wide secret key is a common key that can be used to encrypt all your volumes. Create a cluster wide secret in Kubernetes using `kubectl` command. Use the same 'portworx' namespace on which you've installed Portworx.

```text
NAMESPACE=portworx
kubectl -n ${NAMESPACE} create secret generic px-vol-encryption \
  --from-literal=cluster-wide-secret-key=<value>
```

The cluster wide encryption key resides in the `px-vol-encryption` secret under the `portworx` namespace. 

Provide the cluster wide secret key to Portworx, that acts as the default encryption key for all volumes.

```text
PX_POD=$(kubectl get pods -l name=portworx -n portworx -o jsonpath='{.items[0].metadata.name}')
kubectl exec $PX_POD -n ${NAMESPACE} -- /opt/pwx/bin/pxctl secrets set-cluster-key \
  --secret cluster-wide-secret-key
```
{{<info>}}
**Note:** The cluster wide key is the secret name where the encrypt key exists.  It does not contain the value to encrypt.
{{</info>}}




## Use Kubernetes Secrets with Portworx

{{<homelist series="kubernetes-secret-uses">}}
