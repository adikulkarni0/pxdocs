---
title: Enable Pure1 integration for upgrades
weight: 100
keywords: Troubleshoot, diags, phonehome, callhome
description: Find out how to enable the Portworx Pure1 integration for clusters that were created before Portworx 2.8.0
hidden: true
aliases:
    - /portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/troubleshooting/enable-pure1-upgrades/
---
This page covers enabling Pure1 integration for the following cases:

* You had a running Portworx cluster prior to Portworx 2.8.0, and you are now upgrading to 2.12.0 or later
* You installed Portworx 2.12.0, but did not enable Pure1 integration and want to do so now

## Prerequisites

* Portworx Operator 1.10.0 or later
* A cluster running Portworx 2.12.0 or later using an Operator-based installation
* Outbound access to the internet to allow connection to Pure1

{{<info>}}
**NOTE:** If you're using a daemonset-based Portworx installation, you must migrate to an Operator-based install, then:

* If you did not previously enable telemetry for your daemonset-based installation, upgrade to version 2.12 before enabling Pure1 integration.
* If you previously enabled telemetry for your daemonset-based installation, when you migrate that to an Operator-based installation, telemetry is still enabled.
{{</info>}}

## Enable telemetry

{{<info>}}
**NOTE:** Telemetry is not supported for air-gapped clusters or when using a custom proxy.
{{</info>}}

Portworx supports telemetry in the `StorageCluster` spec, but it's disabled by default. To enable the metrics collector and telemetry integration with Pure1, add the following section in your StorageCluster spec:

```text
spec:
  monitoring:
    telemetry:
      enabled: true 
```

<!--
**CAUTION:** If you're using a custom proxy with telemetry, you must format your proxy URL as `<endpoint>:<port>`. 

For example:

```text
spec:
  env:
  - name: PX_HTTP_PROXY
    value: http://10.7.69.67:8888
```
-->

This adds telemetry components to Portworx and automatically creates the `pure-telemetry-certs` secret in the `kube-system` namespace. You can also enable telemetry from [PX-Central](https://central.portworx.com/specGen/wizard) via **Advanced Settings** on the **Customize** page.

  ![Pure1 Integration](/img/enable-Pure1-integration.png)

