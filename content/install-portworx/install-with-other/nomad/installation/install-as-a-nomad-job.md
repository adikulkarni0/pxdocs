---
title: Install as a Nomad Job
linkTitle: Install as a Nomad job
keywords: Install, nomad job
description: Learn how to install Portworx using Nomad job with CSI enabled.
weight: 100
series: px-as-a-nomad-job
noicon: true
aliases:
    - /install-with-other/nomad/installation/install-as-a-nomad-job/
---
{{<info>}}
This document presents the **Nomad** method of installing a Portworx cluster with CSI enabled. Please refer to the [Portworx on Kubernetes](/operations/operate-kubernetes/) page if you want to install Portworx on Kubernetes.
{{</info>}}

This section shows how to install Portworx using a Nomad job.

## Prerequisites

* Nomad 1.1.0 and Portworx 2.8.0 are the minimum versions needed for most CSI volume operations. 

## Configure your Nomad clients

The Portworx OCI-monitor container needs to run in privileged mode, so you need to configure your Nomad clients to allow docker containers running on privileged mode. 

Add the following lines in your Nomad client configuration files and restart your clients:

```text
plugin "docker" {
  config {
    allow_privileged = true
    volumes {
      enabled      = true
    }
  }
}
```

## Install Portworx 

### Create a Nomad job file

1. Create a new file called `portworx.nomad` with the following content, specifying your own information for the following fields:
   
   * the `datacenters` on the file above to fit your environment
   * the `count` to specify the number of nomad nodes you want Portworx to be installed on
   * Optionally, any constraints you may have

      {{<info>}}
**NOTE:** This example installs Portworx on a Nomad cluster with 3 clients.
      {{</info>}}

    ```text
    job "portworx" {
      type        = "service"
      datacenters = ["dc1"]
  
      group "portworx" {
        count = 3

        constraint {
          operator  = "distinct_hosts"
          value     = "true"
        }

        # restart policy for failed portworx tasks
        restart {
          attempts = 3
          delay    = "30s"
          interval = "5m"
          mode     = "fail"
        }

        # how to handle upgrades of portworx instances
        update {
          max_parallel     = 1
          health_check     = "checks"
          min_healthy_time = "10s"
          healthy_deadline = "5m"
          auto_revert      = true
          canary           = 0
          stagger          = "30s"
        }

        network {
          port "portworx" {
            static = "9015"
          }
        }

        task "px-node" {
          driver = "docker"
          kill_timeout = "120s"   # allow portworx 2 min to gracefully shut down
          kill_signal = "SIGTERM" # use SIGTERM to shut down the nodes

          # setup environment variables for px-nodes
          env {
            AUTO_NODE_RECOVERY_TIMEOUT_IN_SECS = "1500"
            PX_TEMPLATE_VERSION                = "V4"
            CSI_ENDPOINT                       = "unix://var/lib/csi/csi.sock"
          }

          # CSI Driver config
          csi_plugin {
            id                     = "portworx"
            type                   = "monolith"
            mount_dir              = "/var/lib/csi"
            health_timeout         = "30m"                  # Nomad 1.3.2 and later only
            stage_publish_base_dir = "/var/lib/csi/publish" # Nomad 1.3.4 and later only
          } 

          # container config
          config {
            image        = "portworx/oci-monitor:2.11.1"
            network_mode = "host"
            ipc_mode = "host"
            privileged = true
        
            # configure your parameters below
            # do not remove the last parameter (needed for health check)
            args = [
                "-c", "px-cluster-nomadv1",
                "-a",
                "-b",
                "-k", "consul://127.0.0.1:8500",
                "--endpoint", "0.0.0.0:9015"
            ]

            volumes = [
                "/var/cores:/var/cores",
                "/var/run/docker.sock:/var/run/docker.sock",
                "/run/containerd:/run/containerd",
                "/etc/pwx:/etc/pwx",
                "/opt/pwx:/opt/pwx",
                "/proc:/host_proc",
                "/etc/systemd/system:/etc/systemd/system",
                "/var/run/log:/var/run/log",
                "/var/log:/var/log",
                "/var/run/dbus:/var/run/dbus"
            ]

          }

          # resource config
          resources {
            cpu    = 1024
            memory = 2048
          }

        }
      }
    }
    ```

1. Start Portworx as a job:

    ```text
    nomad job run portworx.nomad
    ```


### Enable PX-Security

Portworx uses a secure system secret to communicate with itself and other Portworx nodes, and a shared secret to generate application, volume, user, and admin tokens. 

To enable PX-Security, update the "env" section in your job config.

1. Generate a secure system secret for PX-Security. You can store this in an environment variable temporarily:

    ```text
    PORTWORX_AUTH_SYSTEM_KEY=$(cat /dev/urandom | base64 | fold -w 64 | head -n 1) \
    ```

2. Generate a shared secret. You can store this in an environment variable temporarily:

    ```text
    PORTWORX_AUTH_SHARED_SECRET=$(cat /dev/urandom | base64 | fold -w 64 | head -n 1)
    ```

3. Pick an issuer name for your cluster. This is typically a company name, deployment name, region, or a combination of those. You can store this in an environment variable temporarily:

    ```text
    PORTWORX_AUTH_JWT_ISSUER=portworx-uswest-02-cluster1.io
    ```

4. Populate the values in your `portworx.nomad` job `env` section with the values you generated in the steps above:

    ```text
          # setup environment variables for px-nodes
          env {
            "AUTO_NODE_RECOVERY_TIMEOUT_IN_SECS" = "1500"
            "PX_TEMPLATE_VERSION"                = "V4"
            "PORTWORX_AUTH_SYSTEM_KEY"           = "<PORTWORX_AUTH_SYSTEM_KEY>"
            "PORTWORX_AUTH_JWT_SHAREDSECRET"     = "<PORTWORX_AUTH_SHARED_SECRET>"
            "PORTWORX_AUTH_JWT_ISSUER"           = "<ISSUER_NAME>" 
          }
    ```

5. Deploy Portworx with security enabled:

    ```text
    nomad job run portworx.nomad
    ```

{{<info>}}
**NOTE:** To start using Portworx on Nomad with security enabled, you can follow the steps on [securing your volumes](/operations/operate-other/operate-nomad/security/).
{{</info>}}

## Monitoring the installation process

Check status and wait for all instances to be healthy:

```text
nomad status portworx
```

```output
ID            = portworx
Name          = portworx
Submit Date   = 2019-05-08T00:48:39Z
Type          = service
Priority      = 50
Datacenters   = dc1
Status        = running
Periodic      = false
Parameterized = false

Summary
Task Group  Queued  Starting  Running  Failed  Complete  Lost
portworx    0       0         3        0       0         0

Latest Deployment
ID          = 64d1d011
Status      = successful
Description = Deployment completed successfully

Deployed
Task Group  Auto Revert  Desired  Placed  Healthy  Unhealthy  Progress Deadline
portworx    true         3        3       3        0          2019-05-08T01:00:40Z

Allocations
ID        Node ID   Task Group  Version  Desired  Status   Created   Modified
20a20fd0  e074a6b0  portworx    0        run      running  2m9s ago  23s ago
54f759fa  2299a3b6  portworx    0        run      running  2m9s ago  9s ago
c44ee856  6138409d  portworx    0        run      running  2m9s ago  10s ago
```

You can also follow the logs to wait for Portworx to be ready.

In the example below, I am using the first allocation ID, which is `20a20fd0`:

```text
nomad alloc logs -f 20a20fd0
```

```output
...
@ip-10-1-1-199 portworx[2414]: time="2019-05-07T15:35:18Z" level=info msg="PX is ready on Node: 6f160613-1fe4-45c4-8d04-079f3bde2921. CLI accessible at /opt/pwx/bin/pxctl."
....
```

After that, you can `ssh` to any one of Nomad clients and check the status using `pxctl`:

```text
pxctl status
```

```output
Status: PX is operational
License: Trial (expires in 31 days)
Node ID: 6f160613-1fe4-45c4-8d04-079f3bde2921
    IP: 10.1.1.199
     Local Storage Pool: 1 pool
    POOL    IO_PRIORITY    RAID_LEVEL    USABLE    USED    STATUS    ZONE        REGION
    0    LOW        raid0        50 GiB    4.3 GiB    Online    us-east-2a    us-east-2
    Local Storage Devices: 1 device
    Device    Path        Media Type        Size        Last-Scan
    0:1    /dev/xvdd    STORAGE_MEDIUM_SSD    50 GiB        07 May 19 15:35 UTC
    total            -            50 GiB
Cluster Summary
    Cluster ID: px-cluster-nomadv4
    Cluster UUID: 8e955967-9c74-4465-8014-610c3fe3c0d7
    Scheduler: none
    Nodes: 3 node(s) with storage (3 online)
    IP        ID                    SchedulerNodeName            StorageNode    Used    Capacity    Status    StorageStatus    Version        Kernel        OS
    10.1.1.111    8a003439-d361-4cfa-8e53-2c32f808290e    8a003439-d361-4cfa-8e53-2c32f808290e    Yes        4.3 GiB    50 GiB        Online    Up        2.1.1.0-9df38f7    4.4.0-1079-aws    Ubuntu 16.04.6 LTS
    10.1.1.199    6f160613-1fe4-45c4-8d04-079f3bde2921    6f160613-1fe4-45c4-8d04-079f3bde2921    Yes        4.3 GiB    50 GiB        Online    Up (This node)    2.1.1.0-9df38f7    4.4.0-1079-aws    Ubuntu 16.04.6 LTS
    10.1.2.34    5f51038a-5296-492a-894e-031bab836747    5f51038a-5296-492a-894e-031bab836747    Yes        4.3 GiB    50 GiB        Online    Up        2.1.1.0-9df38f7    4.4.0-1079-aws    Ubuntu 16.04.6 LTS
Global Storage Pool
    Total Used        :  13 GiB
    Total Capacity    :  150 GiB
```

{{<info>}}
**NOTE:**
If Portworx crashes or you stop it on a node, Nomad will currently not detect that Portworx is unhealthy and restart the container.
{{</info>}}

## Post-Install

Once you have successfully installed Portworx using a Nomad job, the following sections might be useful.

{{<homelist series2="px-postinstall-nomad-job">}}
