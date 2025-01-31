---
title: "Monitoring your cluster"
keywords: monitoring, prometheus, alert manager, grafana, datadog
description: How to monitor your Portworx cluster.
weight: 200
aliases:
    - /install-with-other/operate-and-maintain/monitoring/
---
{{<info>}}
This document presents the **non-Kubernetes** method of monitoring a Portworx cluster. Please refer to the [Monitoring](/operations/operate-kubernetes/monitoring/) page if you are running Portworx on Kubernetes.
{{</info>}}

## Using node_exporter and cadvisor alongside Portworx

[Node exporter](https://github.com/prometheus/node_exporter) and [cadvisor with prometheus](https://github.com/google/cadvisor/blob/master/docs/storage/prometheus.md) are great tools that export metrics about hardware/OS and containers. In order to provide metrics from the host, both tools require '/' to be exported into the container as '/rootfs'.

Portworx exports mounts to other containers and these in turn also get exported to node_exporter and cadvisor. In order for all mount events to be propagated to these containers, the root fs on the host should be bind mounted as "ro:slave". If this is not done, it is possible that these containers hold on to these volume mounts preventing the Portworx volumes from being used on other hosts.

```text
# Host root filesystem should be mounted as read-only:slave
-v "/:/rootfs:ro" \
```

## Monitoring Portworx

Portworx exports metrics to Prometheus as are described [here](/operations/operate-other/monitoring/prometheus)

Alert manager integration is described [here](/operations/operate-other/monitoring/alerting)

Grafana templates are provided [here](/operations/operate-other/monitoring/grafana/)

Datadog integration with Portworx is described [here](https://docs.datadoghq.com/integrations/portworx/)
