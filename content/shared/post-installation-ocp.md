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

Enter the following `oc get pods` command to list and filter the results for Portworx pods:

```text 
oc get pods -n portworx -o wide | grep -e portworx -e px
```

```output
portworx-api-774c2                                      1/1     Running   0                2m55s   192.168.121.196   username-k8s1-node0    <none>           <none>
portworx-api-t4lf9                                      1/1     Running   0                2m55s   192.168.121.99    username-k8s1-node1    <none>           <none>
portworx-kvdb-94bpk                                     1/1     Running   0                4s      192.168.121.196   username-k8s1-node0    <none>           <none>
portworx-operator-58967ddd6d-kmz6c                      1/1     Running   0                4m1s    10.244.1.99       username-k8s1-node0    <none>           <none>
prometheus-px-prometheus-0                              2/2     Running   0                2m41s   10.244.1.105      username-k8s1-node0    <none>           <none>
px-cluster-1c3edc42-4541-48fc-b173-3e9bf3cd834d-9gs79   2/2     Running   0                2m55s   192.168.121.196   username-k8s1-node0    <none>           <none>
px-cluster-1c3edc42-4541-48fc-b173-3e9bf3cd834d-vpptx   1/2     Running   0                2m55s   192.168.121.99    username-k8s1-node1    <none>           <none>
px-csi-ext-868fcb9fc6-54bmc                             4/4     Running   0                3m5s    10.244.1.103      username-k8s1-node0    <none>           <none>
px-csi-ext-868fcb9fc6-8tk79                             4/4     Running   0                3m5s    10.244.1.102      username-k8s1-node0    <none>           <none>
px-csi-ext-868fcb9fc6-vbqzk                             4/4     Running   0                3m5s    10.244.3.107      username-k8s1-node1    <none>           <none>
px-prometheus-operator-59b98b5897-9nwfv                 1/1     Running   0                3m3s    10.244.1.104      username-k8s1-node0    <none>           <none>
```

Note the name of one of your `px-cluster` pods. You'll run `pxctl` commands from these pods in following steps. 

### Verify Portworx cluster status

You can find the status of the Portworx cluster by running `pxctl status` commands from a pod. Enter the following `oc exec` command, specifying the pod name you retrieved in the previous section:

```text
oc exec px-cluster-1c3edc42-4541-48fc-b173-3e9bf3cd834d-vpptx -n portworx -- /opt/pwx/bin/pxctl status
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
        Nodes: 2 node(s) with storage (2 online)
        IP              ID                                      SchedulerNodeName       Auth            StorageNode     Used    Capacity        Status  StorageStatus       Version         Kernel                  OS
        192.168.121.196 f6d87392-81f4-459a-b3d4-fad8c65b8edc    username-k8s1-node0      Disabled        Yes             10 GiB  3.0 TiB         Online  Up 2.11.0-81faacc   3.10.0-1127.el7.x86_64  CentOS Linux 7 (Core)
        192.168.121.99  788bf810-57c4-4df1-9a5a-70c31d0f478e    username-k8s1-node1      Disabled        Yes             10 GiB  3.0 TiB         Online  Up (This node)      2.11.0-81faacc  3.10.0-1127.el7.x86_64  CentOS Linux 7 (Core)
Global Storage Pool
        Total Used      :  20 GiB
        Total Capacity  :  6.0 TiB
```

The Portworx status will display `PX is operational` if your cluster is running as intended. 

<!-- I'd love to give them a list of things to review here. What's important?

* look for warnings, if any
* Status: PX is operational
 
-->
### Verify pxctl cluster provision status

* Find the storage cluster, the status should show as `Online`:

    ```text
    oc -n portworx get storagecluster
    ```
    ```output
    NAME                                              CLUSTER UUID                           STATUS   VERSION   AGE
    px-cluster-1c3edc42-4541-48fc-b173-3e9bf3cd834d   33a82fe9-d93b-435b-943e-6f3fd5522eae   Online   2.11.0    10m
    ```

* Find the storage nodes, the statuses should show as `Online`:

    ```text
    oc -n portworx get storagenodes
    ```

    ```output
    NAME                  ID                                     STATUS   VERSION          AGE
    username-k8s1-node0   f6d87392-81f4-459a-b3d4-fad8c65b8edc   Online   2.11.0-81faacc   11m
    username-k8s1-node1   788bf810-57c4-4df1-9a5a-70c31d0f478e   Online   2.11.0-81faacc   11m
    ```       

* Verify the Portworx cluster provision status <!-- What's the success condition here?-->. Enter the following `oc exec` command, specifying the pod name you retrieved in the previous section:

    ```text
    oc exec px-cluster-1c3edc42-4541-48fc-b173-3e9bf3cd834d-vpptx -n portworx -- /opt/pwx/bin/pxctl cluster provision-status
    ```

    ```output
    Defaulted container "portworx" out of: portworx, csi-node-driver-registrar
    NODE                                    NODE STATUS     POOL                                            POOL STATUS     IO_PRIORITY     SIZE    AVAILABLE  USED     PROVISIONED     ZONE    REGION  RACK
    788bf810-57c4-4df1-9a5a-70c31d0f478e    Up              0 ( 96e7ff01-fcff-4715-b61b-4d74ecc7e159 )      Online          HIGH            3.0 TiB 3.0 TiB    10 GiB   0 B             default default default
    f6d87392-81f4-459a-b3d4-fad8c65b8edc    Up              0 ( e06386e7-b769-4ce0-b674-97e4359e57c0 )      Online          HIGH            3.0 TiB 3.0 TiB    10 GiB   0 B             default default default
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
2. Run the `oc apply` command to create a PVC:

    ```text
    oc apply -f <your-pvc-name>.yaml
    ```
    ```output
    persistentvolumeclaim/example-pvc created
    ```

### Verify your StorageClass and PVC

1. Enter the following `oc get storageclass` command, specify the name of the StorageClass you created in the steps above:

    ```text
    oc get storageclass <your-storageclass-name>
    ```
    ```output
    NAME                   PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
    example-storageclass   pxd.portworx.com   Delete          Immediate           false                  24m
    ```

    `oc` will return details about your storageClass if it was created correctly. Verify the configuration details appear as you intended. 

2. Enter the `oc get pvc` command, if this is the only StorageClass and PVC you've created, you should see only one entry in the output:

    ```text
    oc get pvc <your-pvc-name>
    ```
    ```output
    NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS           AGE
    example-pvc   Bound    pvc-dce346e8-ff02-4dfb-935c-2377767c8ce0   2Gi        RWO            example-storageclass   3m7s
    ```

    `oc` will return details about your PVC if it was created correctly. Verify the configuration details appear as you intended. 


<!-- 
## Monitor your Portworx cluster 

You can use the following technologies to monitor your Portworx cluster:

* Prometheus to collect data
* Alertmanager to provide notifications
* Grafana to visualize your data

### Verify monitoring using Prometheus

You can monitor your Portworx cluster using Prometheus. Portworx deploys Prometheus by default, but you can verify the deployment:

1. Verify that Prometheus pods are running by entering the following `oc get pods` command:

    ```text 
    oc get pods -A | grep -i prometheus
    ```
    ```output
    portworx   prometheus-px-prometheus-0                              2/2     Running            0                59m
    portworx   px-prometheus-operator-59b98b5897-9nwfv                 1/1     Running            0                60m
    ```

2. Verify that the Prometheus `px-prometheus` and `prometheus operated` services exist by entering the following `oc get service` command:

    ```text 
    oc -n portworx get service | grep -i prometheus
    ```
    ```output
    prometheus-operated         ClusterIP   None             <none>        9090/TCP                       63m
    px-prometheus               ClusterIP   10.99.61.133     <none>        9090/TCP                       63m
    ```
### Set up Alertmanager

Prometheus Alertmanager handles alerts sent from the Prometheus server based on rules you set. If any Prometheus rule is triggered, Alertmanager sends a corresponding notification to the specified receivers. You can configure these receivers using an Alertmanager config file. Perform the following steps to configure and enable Alertmanager: 

1. Create a valid [Alertmanager configuration](https://prometheus.io/docs/alerting/latest/configuration/) file and name it `alertmanager.yaml`.

1. Create a secret called `alertmanager-portworx` in the same namespace as your StorageCluster object:

    ```text
    oc -n portworx create secret generic alertmanager-portworx --from-file=alertmanager.yaml=alertmanager.yaml
    ```

2. Edit your StorageCluster object to enable Alertmanager:

    ```text
    apiVersion: core.libopenstorage.org/v1
    kind: StorageCluster
    metadata:
      name: portworx
      namespace: portworx
    monitoring:
        prometheus:
          enabled: true
          exportMetrics: true
          alertManager:
            enabled: true
    ```

3. Access Alertmanager by setting up port forwarding and browsing to the specified port. In this example, port forwarding is provided for ease of access to the Alertmanager service from the node IP using the port 9093:

    ```text 
    oc -n portworx port-forward service/alertmanager-portworx 9093:9093
    ```

    The following is a sample for Alertmanager, the settings used in your environment may be different:

    ```text
    global:
      # The smarthost and SMTP sender used for mail notifications.
      smtp_smarthost: 'smtp.gmail.com:587'
      smtp_from: 'abc@test.com'
      smtp_auth_username: "abc@test.com"
      smtp_auth_password: 'xyxsy'
    route:
      group_by: [Alertname]
      # Send all notifications to me.
      receiver: email-me
    receivers:
    - name: email-me
      email_configs:
      - to: abc@test.com
        from: abc@test.com
        smarthost: smtp.gmail.com:587
        auth_username: "abc@test.com"
        auth_identity: "abc@test.com"
        auth_password: "abc@test.com"
    ```

{{<info>}}
**NOTE:** PX-Central on-premises includes Grafana and Portworx dashboards natively, which you can use to monitor your Portworx cluster. Refer to the [PX-Central documentation](https://central.docs.portworx.com/) for further details.
{{</info>}}

### Configure Grafana

You can connect to Prometheus using Grafana to visualize your data. 

1. Enter the following `curl` commands to download the Grafana dashboard and datasource configuration files:

    ```text 
    curl -O https://docs.portworx.com/samples/k8s/pxc/grafana-dashboard-config.yaml
    ```
    ```output
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    100   211  100   211    0     0    596      0 --:--:-- --:--:-- --:--:--   596
    ```

    ```text 
    curl -O https://docs.portworx.com/samples/k8s/pxc/grafana-datasource.yaml
    ```
    ```output
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    100  1625  100  1625    0     0   4456      0 --:--:-- --:--:-- --:--:--  4464
    ```

2. Create a configmap for the dashboard and data source.
    
    ```text 
    oc -n portworx create configmap grafana-dashboard-config --from-file=grafana-dashboard-config.yaml
    ```
    ```text
    oc -n portworx create configmap grafana-source-config --from-file=grafana-datasource.yaml
    ```

3. Download and install Grafana templates using the followin `curl` and `oc` commands:

    ```text
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-cluster-dashboard.json" -o portworx-cluster-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-node-dashboard.json" -o portworx-node-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-volume-dashboard.json" -o portworx-volume-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-performance-dashboard.json" -o portworx-performance-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-etcd-dashboard.json" -o portworx-etcd-dashboard.json && \
    ```

    ```text
    oc -n portworx create configmap grafana-dashboards \
    --from-file=portworx-cluster-dashboard.json \
    --from-file=portworx-performance-dashboard.json \
    --from-file=portworx-node-dashboard.json \
    --from-file=portworx-volume-dashboard.json \
    --from-file=portworx-etcd-dashboard.json 
    ```

4. Enter the following `oc apply` command to download and install Grafana YAML file:

    ```text 
    oc apply -f https://docs.portworx.com/samples/k8s/pxc/grafana.yaml
    ```

5. Edit services to access Prometheus and Grafana through Node Port. You can also use Load Balancer.

    * Login to Grafana using the following credentials:
    
        *Grafana login / password : admin / prom-operator*

    * Run the following command to edit services.

        ```text
        oc -n portworx edit service px-prometheus
        ```

        ```output
        spec:
        clusterIP: 10.109.79.109
        clusterIPs:
        - 10.109.79.109
        internalTrafficPolicy: Cluster
        ipFamilies:
        - IPv4
        ipFamilyPolicy: SingleStack
        ports:
        - name: web
            port: 9090
            protocol: TCP
            targetPort: 9090
        selector:
            prometheus: px-prometheus
        sessionAffinity: None
        type: NodePort
        status:
        loadBalancer: {}
        ```
6. List where the Grafana service is running.

    ```text 
    oc get service -n portworx grafana
    ```

    ```output
    NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    grafana   ClusterIP   10.111.67.112   <none>        3000/TCP   16s
    ```

7. Change “type” under spec to `NodePort`

    ``` text
    oc edit service grafana -n portworx
    service/grafana edited
    ```

    ```text
    oc get service -n portworx grafana
    ```

    ```output
    NAME      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    grafana   NodePort   10.111.67.112   <none>        3000:31507/TCP   65s
    ```

8. Run the following command to list the nodes in your cluster. Note any worker node IP address for future reference.

    ```text 
    oc get nodes -o wide
    ```

    ```output
    NAME       STATUS   ROLES                  AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
    master-4   Ready    control-plane,master   60d   v1.22.8   10.13.21.155   <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://1.13.1
    node-4-1   Ready    <none>                 60d   v1.22.8   10.13.21.142   <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://1.13.1
    node-4-2   Ready    <none>                 60d   v1.22.8   10.13.21.31    <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://1.13.1
    node-4-3   Ready    <none>                 60d   v1.22.8   10.13.21.4     <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://1.13.1
    node-4-4   Ready    <none>                 60d   v1.22.8   10.13.21.95    <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://1.13.1
    node-4-5   Ready    <none>                 60d   v1.22.8   10.13.21.68    <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://1.13.1
    node-4-6   Ready    <none>                 60d   v1.22.8   10.13.21.58    <none>        CentOS Linux 7 (Core)   3.10.0-1160.45.1.el7.x86_64   docker://1.13.1
    ```

9. Navigate to Grafana by specifying your worker IP address and the 31507 port:

    ![Grafana Welcome screen](/img/Post-installation/image2.png)
    
10. Enter the default credentials to login:

    * **login:**  `admin`
    * **password:** `admin`

-->