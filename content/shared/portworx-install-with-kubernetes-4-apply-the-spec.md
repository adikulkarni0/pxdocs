---
title: Shared content for install Portworx with Kubernetes - apply the specs
hidden: true
description: Shared content for install Portworx with Kubernetes - apply the specs
keywords: Install, apply the specs, kubernetes, k8s
---

### Apply specs

Apply the Operator and StorageCluster specs you generated in the section above using the `kubectl apply` command:

1. Deploy the Operator:

    ```
    kubectl apply -f 'https://install.portworx.com/<version-number>?comp=pxoperator'
    ```
    ```output
    serviceaccount/portworx-operator created
    podsecuritypolicy.policy/px-operator created
    clusterrole.rbac.authorization.k8s.io/portworx-operator created
    clusterrolebinding.rbac.authorization.k8s.io/portworx-operator created
    deployment.apps/portworx-operator created
    ```

2. Deploy the StorageCluster:

    ```
    kubectl apply -f 'https://install.portworx.com/<version-number>?operator=true&mc=false&kbver=&b=true&kd=type%3Dgp2%2Csize%3D150&s=%22type%3Dgp2%2Csize%3D150%22&c=px-cluster-xxxxxx-568f471c2c26&eks=true&stork=true&csi=true&mon=true&tel=false&st=k8s&e==AWS_ACCESS_KEY_IDXXXXXXXXXXXXXXAWS_SECRET_ACCESS_KEYXXXXXXXXXXXXXXXXXXXXX&promop=true'
    ```
    ```output
    storagecluster.core.libopenstorage.org/px-cluster-0d8dad46-f9fd-4945-b4ac-8dfd338e915b created
    ```

#####  Monitor the Portworx pods

1. Enter the following `kubectl get` command, waiting until all Portworx pods show as ready in the output:

    ```text
    kubectl get pods -o wide -n kube-system -l name=portworx
    ```

2. Enter the following `kubectl describe` command with the ID of one of your Portworx pods to show the current installation status for individual nodes:

     ```text
     kubectl -n kube-system describe pods <portworx-pod-id>
     ```
     ```output
     Events:
       Type     Reason                             Age                     From                  Message
       ----     ------                             ----                    ----                  -------
       Normal   Scheduled                          7m57s                   default-scheduler     Successfully assigned kube-system/portworx-qxtw4 to k8s-node-2
       Normal   Pulling                            7m55s                   kubelet, k8s-node-2   Pulling image "portworx/oci-monitor:2.5.0"
       Normal   Pulled                             7m54s                   kubelet, k8s-node-2   Successfully pulled image "portworx/oci-monitor:2.5.0"
       Normal   Created                            7m53s                   kubelet, k8s-node-2   Created container portworx
       Normal   Started                            7m51s                   kubelet, k8s-node-2   Started container portworx
       Normal   PortworxMonitorImagePullInPrgress  7m48s                   portworx, k8s-node-2  Portworx image portworx/px-enterprise:2.5.0 pull and extraction in progress
       Warning  NodeStateChange                    5m26s                   portworx, k8s-node-2  Node is not in quorum. Waiting to connect to peer nodes on port 9002.
       Warning  Unhealthy                          5m15s (x15 over 7m35s)  kubelet, k8s-node-2   Readiness probe failed: HTTP probe failed with statuscode: 503
       Normal   NodeStartSuccess                   5m7s                    portworx, k8s-node-2  PX is ready on this node
     ```

     {{<info>}}
**NOTE:** In your output, the image pulled will differ based on your chosen Portworx license type and version.
     {{</info>}}

#####  Monitor the cluster status

Use the `pxctl status` command to display the status of your Portworx cluster:

```text
PX_POD=$(kubectl get pods -l name=portworx -n kube-system -o jsonpath='{.items[0].metadata.name}')
kubectl exec $PX_POD -n kube-system -- /opt/pwx/bin/pxctl status
```
