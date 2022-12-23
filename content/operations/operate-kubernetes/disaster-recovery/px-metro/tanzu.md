---
title: Installing on a Tanzu cluster
weight: 200
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration, tanzu
hidden: true
---

## Installing on a Tanzu cluster

The existing Sync DR implementation assumes the availability of cloud drives for each of the two clusters. In Tanzu cloud drive is a PVC, which is a cluster-scoped resource and in order to help clusters distinguish between their drives,
they should be labeled accordingly.

* Label your worker nodes **on both** clusters with the `px-dr` label. Set your cluster domain as a
  value.

    ```
    kubectl label nodes <list_of_nodes or --all> px-dr=<cluster_domain_name>
    ```

* Add the `--node_pool_label` parameter to the spec with a key `px-dr`

    ```text
          containers:
            - name: portworx
              image: portworx/oci-monitor:2.8
              args:
                ["-k", "etcd:http://100.26.199.167:2379,etcd:http://100.26.199.168:2379,etcd:http://100.26.199.169:2379", "-c", "px-dr-cluster", "-cluster_domain", "us-east-1a", "-a", "-secret_type", "k8s",
                 "-x", "kubernetes", --node_pool_label=px-dr]
    ```

{{<info>}}
**NOTE:** 

* Other arguments are for illustration purposes only, Use args from your existing specs. Only add `--node_pool_label=px-dr`.
* Only deploy Portworx in this mode when your Kubernetes clusters are separated by a metropolitan area network with a maximum latency of 10ms. Pure Storage does not recommended you run in this mode if your Kubernetes clusters are distributed across regions, such as AWS regions `us-east` and `us-west`. 
{{</info>}}

A cluster domain and, in turn, the set of nodes which are a part of that domain are associated with either of the
following states:

* **Active State**: Nodes from an active cluster domain participate in the cluster activities. Applications **can** be
  scheduled on the nodes which are a part of an active cluster domain.
* **Inactive State**: Nodes from an inactive cluster domain do not participate in the cluster activities.
  Applications **cannot** be scheduled on such nodes.

Once the Kubernetes manifest is applied on both the clusters, Portworx will form the stretch cluster. You can run the
following commands to ensure Portworx is successfully installed in a stretch fashion.

**Kubernetes Cluster 1 running in `us-east-1a`**

```text
kubectl get nodes -o wide
```

```output
ip-172-20-33-100.ec2.internal   Ready     node      3h        v1.11.9   18.205.7.13      Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-20-36-212.ec2.internal   Ready     node      3h        v1.11.9   18.213.118.236   Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-20-48-24.ec2.internal    Ready     master    3h        v1.11.9   18.215.34.139    Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-20-59-248.ec2.internal   Ready     node      3h        v1.11.9   34.200.228.223   Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
```

**Kubernetes Cluster 2 running in `us-east-1b`**

```text
kubectl get nodes -o wide
```

```output
ip-172-40-34-140.ec2.internal   Ready     node      3h        v1.11.9   34.204.2.95     Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-40-35-23.ec2.internal    Ready     master    3h        v1.11.9   34.238.44.60    Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-40-40-230.ec2.internal   Ready     node      3h        v1.11.9   52.90.187.179   Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-40-50-47.ec2.internal    Ready     node      3h        v1.11.9   3.84.27.79      Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
```

A single Portworx cluster running across both the Kubernetes clusters

```text
kubectl exec portworx-d6rk7 -n kube-system -- /opt/pwx/bin/pxctl status
```

```output
Status: PX is operational
License: Trial (expires in 31 days)
Node ID: 04de0858-4081-47c3-a2ab-f0835b788738
        IP: 192.0.2.0
        Local Storage Pool: 1 pool
        POOL    IO_PRIORITY     RAID_LEVEL      USABLE  USED    STATUS  ZONE            REGION
        0       LOW             raid0           150 GiB 9.0 GiB Online  us-east-1b      us-east-1
        Local Storage Devices: 1 device
        Device  Path            Media Type              Size            Last-Scan
        0:1     /dev/xvdf       STORAGE_MEDIUM_SSD      150 GiB         09 Apr 19 22:57 UTC
        total                   -                       150 GiB
        Cache Devices:
        No cache devices
Cluster Summary
        Cluster ID: px-dr-cluster
        Cluster UUID: 27558ed9-7ddd-4424-92d4-5dfe2af572e0
        Scheduler: kubernetes
        Nodes: 6 node(s) with storage (6 online)
        IP              ID                                      SchedulerNodeName               StorageNode     Used    Capacity        Status  StorageStatus   Version         Kernel          OS
        172.20.33.100   c665fe35-57d9-4302-b6f7-a978f17cd020    ip-172-20-33-100.ec2.internal   Yes             0 B     150 GiB         Online  Up              2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
        172.40.50.47    bbb2f11d-c6ad-46e7-a52f-530f313869f3    ip-172-40-50-47.ec2.internal    Yes             0 B     150 GiB         Online  Up              2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
        172.40.34.140   a888a08e-0596-43a5-8d02-6faf19e8724c    ip-172-40-34-140.ec2.internal   Yes             0 B     150 GiB         Online  Up              2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
        172.20.36.212   7a83c652-ffaf-452f-978c-82b0da1d2580    ip-172-20-36-212.ec2.internal   Yes             0 B     150 GiB         Online  Up              2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
        172.20.59.248   11e0656a-45a5-4a5b-b4e6-51e130959644    ip-172-20-59-248.ec2.internal   Yes             0 B     150 GiB         Online  Up              2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
        172.40.40.230   04de0858-4081-47c3-a2ab-f0835b788738    ip-172-40-40-230.ec2.internal   Yes             0 B     150 GiB         Online  Up (This node)  2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
Global Storage Pool
        Total Used      :  0 B
        Total Capacity  :  900 GiB
```

Once your cluster is up and running, [set up a witness node](/operations/operate-kubernetes/disaster-recovery/px-metro/1-install-px#setup-a-witness-node)