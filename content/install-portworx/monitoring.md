---
title: Monitor your Portworx cluster
weight: 1000
description: Set up monitoring
keywords: Portworx, monitoring, Prometheus, Alertmanager, Grafana, node-exporter
aliases:
    - /operations/operate-kubernetes/monitoring/alertmanager-operator/
    - /operations/operate-kubernetes/monitoring/monitoring-px-prometheusAndGrafana.1
---

## Prerequisites
* Portworx version 2.11 or later
* Portworx Operator version 1.9.0 or later

## Monitoring function

You can use the following technologies to monitor your Portworx cluster:

* Prometheus to collect data
* Alertmanager to provide notifications
* Grafana to visualize your data

## Verify monitoring using Prometheus

You can monitor your Portworx cluster using Prometheus. Portworx deploys Prometheus by default, but you can verify the deployment:

1. Verify that Prometheus pods are running by entering the following `kubectl get pods` command in the namespace where you deployed Portworx. For example:

    ```text 
    kubectl -n kube-system get pods -A | grep -i prometheus
    ```
    ```output
    kube-system   prometheus-px-prometheus-0                              2/2     Running            0                59m
    kube-system   px-prometheus-operator-59b98b5897-9nwfv                 1/1     Running            0                60m
    ```

2. Verify that the Prometheus `px-prometheus` and `prometheus operated` services exist by entering the following command:

    ```text 
    kubectl -n kube-system  get service | grep -i prometheus
    ```
    ```output
    prometheus-operated         ClusterIP   None             <none>        9090/TCP                       63m
    px-prometheus               ClusterIP   10.99.61.133     <none>        9090/TCP                       63m
    ```
## Set up Alertmanager

Prometheus Alertmanager handles alerts sent from the Prometheus server based on rules you set. If any Prometheus rule is triggered, Alertmanager sends a corresponding notification to the specified receivers. You can configure these receivers using an Alertmanager config file. Perform the following steps to configure and enable Alertmanager: 

1. Create a valid [Alertmanager configuration](https://prometheus.io/docs/alerting/latest/configuration/) file and name it `alertmanager.yaml`. The following is a sample for Alertmanager, and the settings used in your environment may be different:

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

2. Create a secret called `alertmanager-portworx` in the same namespace as your StorageCluster object:

    ```text
    kubectl -n kube-system create secret generic alertmanager-portworx --from-file=alertmanager.yaml=alertmanager.yaml
    ```

3. Edit your StorageCluster object to enable Alertmanager:

    ```text
    kubectl -n kube-system edit stc <px-cluster-name>
    ```

    ```text
    apiVersion: core.libopenstorage.org/v1
    kind: StorageCluster
    metadata:
      name: portworx
      namespace: kube-system
    monitoring:
        prometheus:
          enabled: true
          exportMetrics: true
          alertManager:
            enabled: true
    ```

4. Verify that the Alertmanager pods are running using the following command:

    ```text
    kubectl -n kube-system get pods | grep -i alertmanager
    ```
    ```output
    alertmanager-portworx-0                    2/2     Running   0              4m9s
    alertmanager-portworx-1                    2/2     Running   0              4m9s
    alertmanager-portworx-2                    2/2     Running   0              4m9s
    ```

    **NOTE:** To view the complete list of out-of-the-box default rules, see step 7 below.

### Access the Alertmanager UI

To access the Alertmanager UI and view the Alertmanager Status and alerts, you need to set up port forwarding and browse to the specified port. In this example, port forwarding is provided for ease of access to the Alertmanager service from your local machine using the port 9093.

1. Set up port forwarding:
    ```text 
    kubectl -n kube-system port-forward service/alertmanager-portworx --address=<masternodeIP> 9093:9093
    ```
2. Access Prometheus UI by browsing to `http://<masternodeIP>:9093/#/status`

    ![Alertmanager Status](/img/monitoring/alertmanagerstatus.png)

    **NOTE:** PX-Central on-premises includes Grafana and Portworx dashboards natively, which you can use to monitor your Portworx cluster. Refer to the [PX-Central documentation](https://central.docs.portworx.com/) for further details.

### Access the Prometheus UI

To access the Prometheus UI to view Status, Graph and default Alerts, you also need to set up port forwarding and browse to the specified port. In this example, Port forwarding is provided for ease of access to the Prometheus UI service from your local machine using the port 9090.

1. Set up port forwarding:

    ```text 
    kubectl -n kube-system port-forward service/px-prometheus 9090:9090
    ```
    ![Prometheus Status](/img/monitoring/prometheusstatus.png)

2. Access the Prometheus UI by browsing to `http://localhost:9090/alerts`.

### View provided Prometheus rules

To view the complete list of out-of-the-box default rules used for event notifications, perform the following steps.

1. Get the Prometheus rules:

    ```text 
    kubectl -n kube-system get prometheusrules
    ```
    ```output
    NAME       AGE
    portworx   46d
    ```
2. Save the Prometheus rules to a yaml file:

    ```text 
    kubectl -n kube-system get prometheusrules portworx -o yaml > prometheusrules.yaml
    ```
3. View the contents of the file:
    ```text 
    cat prometheusrules.yaml
    ```

## Configure Grafana

You can connect to Prometheus using Grafana to visualize your data. Grafana is a multi-platform open source analytics and interactive visualization web application. It provides charts, graphs, and alerts.

1. Enter the following commands to download the Grafana dashboard and datasource configuration files:

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

2. Create a configmap for the dashboard and data source:
    
    ```text 
    kubectl -n kube-system create configmap grafana-dashboard-config --from-file=grafana-dashboard-config.yaml
    ```
    ```text
    kubectl -n kube-system create configmap grafana-source-config --from-file=grafana-datasource.yaml
    ```

3. Download and install Grafana dashboards using the following commands:

    ```text
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-cluster-dashboard.json" -o portworx-cluster-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-node-dashboard.json" -o portworx-node-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-volume-dashboard.json" -o portworx-volume-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-performance-dashboard.json" -o portworx-performance-dashboard.json && \
    curl "https://docs.portworx.com/samples/k8s/pxc/portworx-etcd-dashboard.json" -o portworx-etcd-dashboard.json
    ```

    ```text
    kubectl -n kube-system create configmap grafana-dashboards \
    --from-file=portworx-cluster-dashboard.json \
    --from-file=portworx-performance-dashboard.json \
    --from-file=portworx-node-dashboard.json \
    --from-file=portworx-volume-dashboard.json \
    --from-file=portworx-etcd-dashboard.json 
    ```

4. Enter the following command to download and install the Grafana YAML file:

    ```text 
    kubectl apply -f https://docs.portworx.com/samples/k8s/pxc/grafana.yaml
    ```

5. Verify if the Grafana pod is running using the following command:

    ```text
    kubectl -n kube-system get pods | grep -i grafana
    ```
    ```output
    grafana-7d789d5cf9-bklf2                   1/1     Running   0              3m12s
    ```

6. Access Grafana by setting up port forwarding and browsing to the specified port. In this example, port forwarding is provided for ease of access to the Grafana service from your local machine using the port 3000:

    ```text
    kubectl -n kube-system port-forward service/grafana 3000:3000
    ```

7. Navigate to Grafana by browsing to `http://localhost:3000`.
    
8. Enter the default credentials to log in.<br><br>

    * **login:**  `admin`
    * **password:** `admin`

    ![Grafana Dashboard](/img/monitoring/grafanadashboard.png)


## Install Node Exporter

After you have configured Grafana, install the Node Exporter binary. It is required for Grafana to measure various machine resources such as, memory, disk, and CPU utilization. The following DaemonSet will be running in the `kube-system` namespace. 

{{<info>}}
**NOTE:** The examples below use the `kube-system` namespace, you should update this to the correct namespace for your environment. Be sure to install in the same namespace where Prometheus and Grafana are running.
{{</info>}}

1. Install node-exporter via DaemonSet by creating a YAML file named `node-exporter.yaml`:

    ```text
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      labels:
        app.kubernetes.io/component: exporter
        app.kubernetes.io/name: node-exporter
      name: node-exporter
      namespace: kube-system
    spec:
      selector:
        matchLabels:
          app.kubernetes.io/component: exporter
          app.kubernetes.io/name: node-exporter
      template:
        metadata:
          labels:
            app.kubernetes.io/component: exporter
            app.kubernetes.io/name: node-exporter
        spec:
          containers:
          - args:
            - --path.sysfs=/host/sys
            - --path.rootfs=/host/root
            - --no-collector.wifi
            - --no-collector.hwmon
            - --collector.filesystem.ignored-mount-points=^/(dev|proc|sys|var/lib/docker/.+|var/lib/kubelet/pods/.+)($|/)
            - --collector.netclass.ignored-devices=^(veth.*)$
            name: node-exporter
            image: prom/node-exporter
            ports:
              - containerPort: 9100
                protocol: TCP
            resources:
              limits:
                cpu: 250m
                memory: 180Mi
              requests:
                cpu: 102m
                memory: 180Mi
            volumeMounts:
            - mountPath: /host/sys
              mountPropagation: HostToContainer
              name: sys
              readOnly: true
            - mountPath: /host/root
              mountPropagation: HostToContainer
              name: root
              readOnly: true
          volumes:
          - hostPath:
              path: /sys
            name: sys
          - hostPath:
              path: /
            name: root
    ```

2. Apply the object using the following command:

    ```text 
    kubectl apply -f node-exporter.yaml -n kube-system
    ```
    ```output
    daemonset.apps/node-exporter created
    ```
### Create a service

Kubernetes service will connect a set of pods to an abstracted service name and IP address. The service provides discovery and routing between the pods. The following service will be called `node-exportersvc.yaml`, and it will use port `9100`.


1. Create the object file and name it `node-exportersvc.yaml`:

    ```text
    ---
    kind: Service
    apiVersion: v1
    metadata:
      name: node-exporter
      namespace: kube-system
      labels:
        name: node-exporter
    spec:
      selector:
          app.kubernetes.io/component: exporter
          app.kubernetes.io/name: node-exporter
      ports:
      - name: node-exporter
        protocol: TCP
        port: 9100
        targetPort: 9100
    ```
2.  Create the service by running the following command:

    ```text 
    kubectl apply -f node-exportersvc.yaml -n kube-system
    ```
    ```output
    service/node-exporter created
    ```

### Create a service monitor

The Service Monitor will scrape the metrics using the following `matchLabels` and endpoint. 


1. Create the object file and name it `node-exporter-svcmonitor.yaml`:

    ```text
    apiVersion: monitoring.coreos.com/v1
    kind: ServiceMonitor
    metadata:
      name: node-exporter
      labels:
        prometheus: portworx
    spec:
      selector:
        matchLabels:
          name: node-exporter
      endpoints:
      - port: node-exporter
    ```


2.  Create the `ServiceMonitor` object by running the following command:

    ```text 
    kubectl apply -f node-exporter-svcmonitor.yaml -n kube-system
    ```
    ```output
    servicemonitor.monitoring.coreos.com/node-exporter created
    ```
3. Verify that the `prometheus` object has the following `serviceMonitorSelector:` appended:

    ```text 
    kubectl get prometheus -n kube-system -o yaml
    ```
    ```output
        serviceMonitorSelector:
      matchExpressions:
      - key: prometheus
        operator: In
        values:
        - portworx
        - px-backup
    ```


The `serviceMonitorSelector` object is automatically appended to the `prometheus` object that is deployed by the Portworx Operator.
The `ServiceMonitor` will match any `serviceMonitor` that has the key `prometheus` and value of `portworx` or `backup`

### View Node Exporter dashboard in Grafana

Log in to the Grafana UI, from **Dashboards** navigate to the **Manage** section, and select **Portworx Performance Monitor**. You can see the dashboards with **(Node Exporter)**:

![Grafana UI](/img/grafana-node-exporter.png)
