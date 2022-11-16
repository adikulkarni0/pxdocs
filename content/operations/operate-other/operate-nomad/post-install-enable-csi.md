---
title: Enable Portworx CSI using a Nomad job
linkTitle: Enable CSI
keywords: post-install, upgrade, Nomad, csi
description: Learn how to enable Portworx CSI using a Nomad job
weight: 100
series: px-postinstall-nomad-job
noicon: true
aliases:
    - /install-with-other/nomad/upgrade-and-maintain/post-install-enable-csi/
---
Enable Portworx CSI with Nomad by editing the `portworx.nomad` file you used for initial deployment and make the following adjustments. After following these steps, your `portworx.nomad` file should look like the example in the [nomad job installation guide](/install-portworx/install-with-other/nomad/installation/install-as-a-nomad-job/).

1. Add the `csi_plugin` stanza to Nomad job specification inside of the "portworx" group:

    ```text
    csi_plugin {
        id                     = "portworx"
        type                   = "monolith"
        mount_dir              = "/var/lib/csi"
        health_timeout         = "30m"                  # Nomad 1.3.2 and later only
        stage_publish_base_dir = "/var/lib/csi/publish" # Nomad 1.3.4 and later only
    }
    ```

2. Add the following environment variable under the "env" stanza:

    ```text
    CSI_ENDPOINT = "unix:///var/lib/csi/csi.sock"
    ```

3. Rerun the `portworx.nomad` file:

    ```text
    nomad run portworx.nomad
    ```

    Nomad will perform a rolling upgrade; only one instance will be updated at a time. This will not impact any applications, assuming they're running with more than one volume replica.

4. Once the CSI plugin is finished starting up, it should show all nodes as healthy:

    ```
    $ nomad plugin status
    Container Storage Interface
    ID        Provider          Controllers Healthy/Expected  Nodes Healthy/Expected
    portworx  pxd.portworx.com  4/4                           4/4
    ```
