---
title: Portworx integration with Prometheus
keywords: monitoring, prometheus, graph, stats
meta-description: Looking to integrate Portworx with Prometheus? Learn to integrate Portworx storage with Prometheus for monitoring today!
aliases:
    - /install-with-other/operate-and-maintain/monitoring/prometheus/index/
---
{{<info>}}
This document presents the **non-Kubernetes** method of monitoring your Portworx cluster with Prometheus. Please refer to the [Prometheus and Grafana](/install-portworx/monitoring) page if you are running Portworx on Kubernetes.
{{</info>}}


Portworx storage and network stats can easily be integrated with [**prometheus**](https://prometheus.io) or similar applications.
These stats are exported at port 9001; your application can poll http://&lt;IP_ADDRESS&gt;:9001/metrics to get their runtime values.

## Integration with Prometheus

### Step 1: Configuring Prometheus to watch a Portworx node
Add your Portworx node as a target in Prometheus config file:

![Prometheus Config File](/img/prometheus-config.png "Prometheus Config File")

In the example above, our node has IP address of 54.173.138.1, so Prometheus is watching 54.173.138.1:9001 as its target. This can be any node in the Portworx cluster.

### Step 2: Portworx metrics to watch and building graphs with Prometheus

Once Prometheus starts watching your Portworx node, you will be able to see new Portworx related metrics added to Prometheus.

![Portworx Metrics in Prometheus](/img/px-metrics-in-prometheus.png "PX Metrics in Prometheus")

You can now build graphs:

![Building a Graph with Prometheus](/img/building-a-graph-with-prometheus.png "Building a Graph with Prometheus")

{{<info>}}**NOTE:** You can also see the stats by running the following curl command:

```text
curl http://<hostname-or-ip>:9001/metrics
```
{{</info>}}

## Related topics

For information on the available Portworx metrics, refer to the [Portworx metrics for monitoring reference](/reference/metrics/).
