---
title: "Migration with Stork on EKS"
keywords: cloud, backup, restore, snapshot, DR, migration, migration, eks
description: How to migrate stateful applications to EKS
hidden: true
aliases:
    - /portworx-install-with-kubernetes/migration/eks/
---
Pairing with an EKS cluster requires the following additional steps because you
also need to pass in your AWS credentials which will be used to generate the IAM token.

## Create a Secret with your AWS credentials
On the source cluster, create a secret in kube-system namespace with your aws credentials
file:

```text
kubectl create secret generic --from-file=$HOME/.aws/credentials -n  kube-system aws-creds
```

```output
secret/aws-creds created
```

## Pass the Secret to Stork

### When Stork is deployed through the Operator

The credentials created in the previous step need to be provided to Stork. When deployed through Portworx Operator, add the following to the `stork` section of the StorageCluster spec:

```text
  stork:
    enabled: true    
    volumes:
    - name: aws-creds
      mountPath: /root/.aws/
      readOnly: true
      secret:
        secretName: aws-creds
```

### When Stork is deployed using the Portworx DaemonSet model

Mount the secret created above in the Stork deployment by performing the following steps.

1. Run the following command to make updates:

    ```text
    kubectl edit deployment -n kube-system stork
    ```

1. Add the following under `spec.template.spec`:

    ```text
    volumes:
    - name: aws-creds
      secret:
           secretName: aws-creds
    ```

1. Add the following under `spec.template.spec.containers`:

    ```text
    volumeMounts:
    - mountPath: /root/.aws/
      name: aws-creds
      readOnly: true
    ```

1. Save the changes and wait for all the Stork pods to be in running state after applying the changes:

    ```text
    kubectl get pods -n kube-system -l name=stork
    ```
