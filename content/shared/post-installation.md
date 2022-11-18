---
title: "Post-installation"
weight: 100
disableprevnext: true
equalizeTableWidths: "50%"
description: Ensuring Portworx is functional post installation
keywords: Portworx, containers, storage
---

## Verify your Portworx installation 

Once you've installed Portworx, you can perform the following tasks to verify that Portworx has installed correctly. 

### Verify if all pods are running

Enter the following `kubectl get pods` command to list and filter the results for Portworx pods:

```text 
kubectl get pods -n kube-system -o wide | grep -e portworx -e px
```

```output
portworx-api-774c2                                      1/1     Running   0                2m55s   192.168.121.196   username-k8s1-node0    <none>           <none>
portworx-api-t4lf9                                      1/1     Running   0                2m55s   192.168.121.99    username-k8s1-node1    <none>           <none>
portworx-api-dvw64                                      1/1     Running   0                2m55s   192.168.121.99    username-k8s1-node2    <none>           <none>
portworx-kvdb-94bpk                                     1/1     Running   0                4s      192.168.121.196   username-k8s1-node0    <none>           <none>
portworx-kvdb-8b67l                                     1/1     Running   0                10s     192.168.121.196   username-k8s1-node1    <none>           <none>
portworx-kvdb-fj72p                                     1/1     Running   0                30s     192.168.121.196   username-k8s1-node2    <none>           <none>
portworx-operator-58967ddd6d-kmz6c                      1/1     Running   0                4m1s    10.244.1.99       username-k8s1-node0    <none>           <none>
prometheus-px-prometheus-0                              2/2     Running   0                2m41s   10.244.1.105      username-k8s1-node0    <none>           <none>
px-cluster-1c3edc42-4541-48fc-b173-3e9bf3cd834d-9gs79   2/2     Running   0                2m55s   192.168.121.196   username-k8s1-node0    <none>           <none>
px-cluster-1c3edc42-4541-48fc-b173-3e9bf3cd834d-vpptx   2/2     Running   0                2m55s   192.168.121.99    username-k8s1-node1    <none>           <none>
px-cluster-1c3edc42-4541-48fc-b173-3e9bf3cd834d-bxmpn   2/2     Running   0                2m55s   192.168.121.191   username-k8s1-node2    <none>           <none>
px-csi-ext-868fcb9fc6-54bmc                             4/4     Running   0                3m5s    10.244.1.103      username-k8s1-node0    <none>           <none>
px-csi-ext-868fcb9fc6-8tk79                             4/4     Running   0                3m5s    10.244.1.102      username-k8s1-node2    <none>           <none>
px-csi-ext-868fcb9fc6-vbqzk                             4/4     Running   0                3m5s    10.244.3.107      username-k8s1-node1    <none>           <none>
px-prometheus-operator-59b98b5897-9nwfv                 1/1     Running   0                3m3s    10.244.1.104      username-k8s1-node0    <none>           <none>
```

Note the name of one of your `px-cluster` pods. You'll run `pxctl` commands from these pods in following steps. 

### Verify Portworx cluster status

You can find the status of the Portworx cluster by running `pxctl status` commands from a pod. Enter the following `kubectl exec` command, specifying the pod name you retrieved in the previous section:

```text
kubectl exec <pod-name> -n kube-system -- /opt/pwx/bin/pxctl status
```
```output
Defaulted container "portworx" out of: portworx, csi-node-driver-registrar
Status: PX is operational
Telemetry: Disabled or Unhealthy
Metering: Disabled or Unhealthy
License: Trial (expires in 31 days)
Node ID: 788bf810-57c4-4df1-9a5a-70c31d0f478e
        IP: 192.168.121.99 
        Local Storage Pool: 1 pool
        POOL    IO_PRIORITY     RAID_LEVEL      USABLE  USED    STATUS  ZONE    REGION
        0       HIGH            raid0           3.0 TiB 10 GiB  Online  default default
        Local Storage Devices: 3 devices
        Device  Path            Media Type              Size            Last-Scan
        0:1     /dev/vdb        STORAGE_MEDIUM_MAGNETIC 1.0 TiB         14 Jul 22 22:03 UTC
        0:2     /dev/vdc        STORAGE_MEDIUM_MAGNETIC 1.0 TiB         14 Jul 22 22:03 UTC
        0:3     /dev/vdd        STORAGE_MEDIUM_MAGNETIC 1.0 TiB         14 Jul 22 22:03 UTC
        * Internal kvdb on this node is sharing this storage device /dev/vdc  to store its data.
        total           -       3.0 TiB
        Cache Devices:
         * No cache devices
Cluster Summary
        Cluster ID: px-cluster-1c3edc42-4541-48fc-b173-3e9bf3cd834d
        Cluster UUID: 33a82fe9-d93b-435b-943e-6f3fd5522eae
        Scheduler: kubernetes
        Nodes: 3 node(s) with storage (3 online)
        IP              ID                                      SchedulerNodeName       Auth            StorageNode     Used    Capacity        Status  StorageStatus       Version         Kernel                  OS
        192.168.121.196 f6d87392-81f4-459a-b3d4-fad8c65b8edc    username-k8s1-node0      Disabled        Yes             10 GiB  3.0 TiB         Online  Up 2.11.0-81faacc   3.10.0-1127.el7.x86_64  CentOS Linux 7 (Core)
        192.168.121.99  788bf810-57c4-4df1-9a5a-70c31d0f478e    username-k8s1-node1      Disabled        Yes             10 GiB  3.0 TiB         Online  Up (This node)      2.11.0-81faacc  3.10.0-1127.el7.x86_64  CentOS Linux 7 (Core)
        192.168.121.191 a8c76018-43d7-4a58-3d7b-19d45b4c541a    username-k8s1-node2      Disabled        Yes             10 GiB  3.0 TiB         Online  Up  2.11.0-81faacc  3.10.0-1127.el7.x86_64  CentOS Linux 7 (Core)
Global Storage Pool        
        Total Used      :  30 GiB
        Total Capacity  :  9.0 TiB
```

The Portworx status will display `PX is operational` if your cluster is running as intended. 

<!-- I'd love to give them a list of things to review here. What's important?

* look for warnings, if any
* Status: PX is operational
 
-->
### Verify pxctl cluster provision status

* Find the storage cluster, the status should show as `Online`:

    ```text
    kubectl -n kube-system get storagecluster
    ```
    ```output
    NAME                                              CLUSTER UUID                           STATUS   VERSION   AGE
    px-cluster-1c3edc42-4541-48fc-b173-3e9bf3cd834d   33a82fe9-d93b-435b-943e-6f3fd5522eae   Online   2.11.0    10m
    ```

* Find the storage nodes, the statuses should show as `Online`:

    ```text
    kubectl -n kube-system get storagenodes
    ```

    ```output
    NAME                  ID                                     STATUS   VERSION          AGE
    username-k8s1-node0   f6d87392-81f4-459a-b3d4-fad8c65b8edc   Online   2.11.0-81faacc   11m
    username-k8s1-node1   788bf810-57c4-4df1-9a5a-70c31d0f478e   Online   2.11.0-81faacc   11m
    username-k8s1-node2   a8c76018-43d7-4a58-3d7b-19d45b4c541a   Online   2.11.0-81faacc   11m
    ```       

* Verify the Portworx cluster provision status. Enter the following `kubectl exec` command, specifying the pod name you retrieved in the previous section:

    ```text
    kubectl exec <pod-name> -n kube-system -- /opt/pwx/bin/pxctl cluster provision-status
    ```

    ```output
    Defaulted container "portworx" out of: portworx, csi-node-driver-registrar
    NODE                                    NODE STATUS     POOL                                            POOL STATUS     IO_PRIORITY     SIZE    AVAILABLE  USED     PROVISIONED     ZONE    REGION  RACK
    788bf810-57c4-4df1-9a5a-70c31d0f478e    Up              0 ( 96e7ff01-fcff-4715-b61b-4d74ecc7e159 )      Online          HIGH            3.0 TiB 3.0 TiB    10 GiB   0 B             default default default
    f6d87392-81f4-459a-b3d4-fad8c65b8edc    Up              0 ( e06386e7-b769-4ce0-b674-97e4359e57c0 )      Online          HIGH            3.0 TiB 3.0 TiB    10 GiB   0 B             default default default
    a8c76018-43d7-4a58-3d7b-19d45b4c541a    Up              0 ( a2e0af91-bb02-1574-611b-8904cab0e019 )      Online          HIGH            3.0 TiB 3.0 TiB    10 GiB   0 B             default default default
    ```

## Create your first PVC

For your apps to use persistent volumes powered by Portworx, you must use a StorageClass that references Portworx as the provisioner. Portworx includes a number of default StorageClasses, which you can reference with PersistentVolumeClaims (PVCs) you create. For a more general overview of how storage works within Kubernetes, refer to the [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) section of the Kubernetes documentation.

Perform the following steps to create a PVC:

1. Create a PVC referencing the `px-csi-db` default StorageClass and save the file:

    ```text
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
        name: px-check-pvc
    spec:
        storageClassName: px-csi-db
        accessModes:
            - ReadWriteOnce
        resources:
            requests:
                storage: 2Gi
    ```

2. Run the `kubectl apply` command to create a PVC:

    ```text
    kubectl apply -f <your-pvc-name>.yaml
    ```
    ```output
    persistentvolumeclaim/example-pvc created
    ```

### Verify your StorageClass and PVC

1. Enter the `kubectl get storageclass` command:

    ```text
    kubectl get storageclass
    ```
    ```output
    NAME                                 PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
    px-csi-db                            pxd.portworx.com                Delete          Immediate           true                   43d
    px-csi-db-cloud-snapshot             pxd.portworx.com                Delete          Immediate           true                   43d
    px-csi-db-cloud-snapshot-encrypted   pxd.portworx.com                Delete          Immediate           true                   43d
    px-csi-db-encrypted                  pxd.portworx.com                Delete          Immediate           true                   43d
    px-csi-db-local-snapshot             pxd.portworx.com                Delete          Immediate           true                   43d
    px-csi-db-local-snapshot-encrypted   pxd.portworx.com                Delete          Immediate           true                   43d
    px-csi-replicated                    pxd.portworx.com                Delete          Immediate           true                   43d
    px-csi-replicated-encrypted          pxd.portworx.com                Delete          Immediate           true                   43d
    px-db                                kubernetes.io/portworx-volume   Delete          Immediate           true                   43d
    px-db-cloud-snapshot                 kubernetes.io/portworx-volume   Delete          Immediate           true                   43d
    px-db-cloud-snapshot-encrypted       kubernetes.io/portworx-volume   Delete          Immediate           true                   43d
    px-db-encrypted                      kubernetes.io/portworx-volume   Delete          Immediate           true                   43d
    px-db-local-snapshot                 kubernetes.io/portworx-volume   Delete          Immediate           true                   43d
    px-db-local-snapshot-encrypted       kubernetes.io/portworx-volume   Delete          Immediate           true                   43d
    px-replicated                        kubernetes.io/portworx-volume   Delete          Immediate           true                   43d
    px-replicated-encrypted              kubernetes.io/portworx-volume   Delete          Immediate           true                   43d
    stork-snapshot-sc                    stork-snapshot                  Delete          Immediate           true                   43d
    ```

    `kubectl` returns details about the StorageClasses available to you. Verify that `px-csi-db` appears in the list. 

2. Enter the `kubectl get pvc` command. If this is the only StorageClass and PVC that you've created, you should see only one entry in the output:

    ```text
    kubectl get pvc <your-pvc-name>
    ```
    ```output
    NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS           AGE
    example-pvc   Bound    pvc-dce346e8-ff02-4dfb-935c-2377767c8ce0   2Gi        RWO            example-storageclass   3m7s
    ```

    `kubectl` returns details about your PVC if it was created correctly. Verify that the configuration details appear as you intended. 


