---
title: Grafana with Portworx
keywords: monitoring, portworx, grafana
meta-description: Find templates for displaying Portworx cluster information within Grafana.
aliases:
    - /install-with-other/operate-and-maintain/monitoring/grafana/index/
---
{{<info>}}
This document presents the **non-Kubernetes** method of monitoring your Portworx cluster with Grafana. Please refer to the [Prometheus and Grafana](/install-portworx/monitoring) page if you are running Portworx on Kubernetes.
{{</info>}}


## Prerequisites

* Some of the panels in the Portworx dashboards require [Node Exporter](https://grafana.com/grafana/dashboards/1860) running on Portworx nodes. Without Node Exporter, these panels might show ‘No data’.
* The [Prometheus AlertManager plugin](https://grafana.com/grafana/plugins/camptocamp-prometheus-alertmanager-datasource/) must be installed on Grafana, and the AlertManager endpoint must be added as a data source.

## Configure Grafana

1. Start Grafana with the following docker run command

      ```text
      docker run --restart=always --name grafana -d -p 3000:3000 grafana/grafana
      ```

2. Log in to Grafana at `http://your_ip_address:3000` in your browser. The default Grafana login is `admin/admin`.

3. Once logged in, Grafana will ask you to configure your datastore. Use the Prometheus that you configured earlier. To use the templates that are provided later, name your datastore 'prometheus'. In the screen below:

      * Choose 'Prometheus' from the 'Type' dropdown.
      * Name datastore 'prometheus'
      * Add URL of your prometheus UI under Http settings -&gt; Url
      * Select **Save & Test**

      ![Grafana data store File](/img/grafana_datastore.png "Grafana data store File")

4. Import the Portworx provided [Cluster](/samples/k8s/pxc/portworx-cluster-dashboard.json), [Volume](/samples/k8s/pxc/portworx-volume-dashboard.json), [Node](/samples/k8s/pxc/portworx-node-dashboard.json), and [Performance](/samples/k8s/pxc/portworx-performance-dashboard.json) Grafana templates: From the dropdown on left in your Grafana dashboard, select **Dashboards** followed by **Import**, and add the cluster, volume, and node templates.

      {{<info>}}
**NOTES:**

* The Performance dashboard should be run only on systems without memory constraints. The dashboard plots a large dataset range, so it's not recommended for systems that have restricted memory.
* Some clusters running with caching enabled might not show any data in I/O Rate.
      {{</info>}}

      Once added, you can view your dashboards:

      ![Grafana cluster status dashboard](/img/grafana-with-px-cluster-dashboard.png)

      ![Grafana volume status dashboard](/img/grafana-with-px-volume-dashboard-01.png)

      ![Grafana node status dashboard](/img/grafana-with-px-volume-dashboard-02.png)

      ![Grafana Portworx node dashboard](/img/grafana-with-px-node-dashboard.png)

<!--
are these the same as what's linked through GitHub above? If so, we should probably just show them using one method or the other.
[Andrei, 2019-12-17]: Don't know but I'm moving them under the `static` folder
## Cluster Template for Grafana
Use [this template](/samples/non-k8s/grafana/Cluster_Template.json) to display Portworx cluster details in Grafana

## Volume Template for Grafana
Use [this template](/samples/non-k8s/grafana/Volume_Template.json) to display Portworx volume details in Grafana
-->
