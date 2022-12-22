---
title: Prepare your environment
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration
description: Prepare your source and destination clusters for asynchronous DR.
series: setup-AysncDR
weight: 200
---

Follow the instructions on this page to prepare your environment.


## Install storkctl

<!-- Is this done on your local environment? -->

Perform the following steps to download `storkctl` from the Stork pod to the system where you are running `kubectl`:

* Linux:

    ```text
    STORK_POD=$(kubectl get pods -n <namespace> -l name=stork -o jsonpath='{.items[0].metadata.name}') &&
    kubectl cp -n kube-system $STORK_POD:/storkctl/linux/storkctl ./storkctl
    sudo mv storkctl /usr/local/bin &&
    sudo chmod +x /usr/local/bin/storkctl
    ```
* OS X:

    ```text
    STORK_POD=$(kubectl get pods -n <namespace> -l name=stork -o jsonpath='{.items[0].metadata.name}') &&
    kubectl cp -n <namespace> $STORK_POD:/storkctl/darwin/storkctl ./storkctl
    sudo mv storkctl /usr/local/bin &&
    sudo chmod +x /usr/local/bin/storkctl
    ```

* Windows:

    1. Copy `storkctl.exe` from the Stork pod:

        ```text
        STORK_POD=$(kubectl get pods -n <namespace> -l name=stork -o jsonpath='{.items[0].metadata.name}') &&
        kubectl cp -n <namespace> $STORK_POD:/storkctl/windows/storkctl.exe ./storkctl.exe
        ```

    2. Move `storkctl.exe` to a directory in your PATH


## Enable load balancing on cloud clusters

If you're running Kubernetes in the cloud, you must configure an External LoadBalancer (ELB) for the Portworx API service on your source and destination clusters.

{{<info>}}
**Warning:** Do not enable load balancing if authorization is not enabled in the Portworx cluster. 
{{</info>}}

Enable load balancing by entering the `kubectl edit service` command and changing the service type value from `nodePort` to `LoadBalancer`:

```text
kubectl edit service portworx-service -n <namespace>
```
```output
kind: Service
apiVersion: v1
metadata:
  name: portworx-service
  namespace: <namespace>
  labels:
    name: portworx
spec:
  selector:
    name: portworx
  type: LoadBalancer
```