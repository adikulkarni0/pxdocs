---
title: Decommission a Node
weight: 500
keywords: Uninstall, decomission a node, Kubernetes, k8s
description: Steps to decommission a Portworx node in your Kubernetes clusters.
series: k8s-uninstall
aliases:
    - /portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/uninstall/decommission-a-node/
---
This guide describes a recommended workflow for decommissioning a Portworx node in your Kubernetes cluster.

{{<info>}}
**NOTE:** The following steps don't apply if you're using an auto-scaling group (ASG) to manage your Portworx nodes. For details about how you can change the size of your auto-scaling group, see the [Scaling the Size of Your Auto Scaling Group](https://docs.aws.amazon.com/autoscaling/ec2/userguide/scaling_plan.html) page of the AWS documentation.
{{</info>}}

## Migrate application pods using Portworx volumes that are running on this node

If you plan to remove Portworx from a node, applications running on that node using Portworx need to be migrated. If Portworx is not running, existing application containers will end up with read-only volumes and new ones will fail to start.

Perform the following steps to migrate select pods.

1. Cordon the node using the following command:

    ```text
    kubectl cordon <node>
    ```

2. Reschedule application pods using Portworx volumes on different nodes:

    ```text
    kubectl delete pod <pod-name> -n <application namespace>
    ```

   Since application pods are expected to be managed by a controller like `Deployment` or `StatefulSet`, Kubernetes will spin up a new replacement pod on another node.

## Decommission Portworx

To decommission Portworx, perform the following steps.

### Remove Portworx from the cluster

Follow [this guide](/operations/operate-other/scaling/scale-down) section "Removing offline Nodes" or "Removing a functional node from a cluster" to decommission the Portworx node from the cluster.

### Remove Portworx installation from the node

Apply the _px/enabled=remove_ label and it will remove the existing Portworx systemd service. It will also apply the _px/enabled=false_ label to stop Portworx from running in future.

For example, below command will remove existing Portworx installation from _minion2_ and also ensure that Portworx pod doesn’t run there in future.

```text
kubectl label nodes < node > px/enabled=remove --overwrite
```

{{<info>}}
**Decommission from Kubernetes:**
If the plan is to decommission this node altogether from the Kubernetes cluster, no further steps are needed.
{{</info>}}

## Ensure application pods using Portworx don’t run on this node

If you need to continue using the Kubernetes node without Portworx, you will need to ensure your application pods using Portworx volumes don’t get scheduled here.

You can ensure this by adding the `schedulerName: stork` field to your application specs (Deployment, Statefulset, etc). Stork is a scheduler extension that will schedule pods using Portworx PVCs only on nodes that have Portworx running. Refer to the [Using scheduler convergence](/operations/operate-kubernetes/storage-operations/hyperconvergence/#using-scheduler-convergence) article for more information.

Another way to achieve this is to use [inter-pod affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#inter-pod-affinity-and-anti-affinity-beta-feature)

* You need to define a pod affinity rule in your applications that ensure that application pods get scheduled only on nodes where the Portworx pod is running.
* Consider the following nginx example:

      ```text
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      selector:
        matchLabels:
          app: nginx
      replicas: 1
      template:
        metadata:
          labels:
            app: nginx
        spec:
          affinity:
            # Inter-pod affinity rule restricting nginx pods to run only on nodes where Portworx pods are running (Portworx pods have a label
            # name=portworx which is used in the rule)
            podAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
              - labelSelector:
                  matchExpressions:
                  - key: name
                    operator: In
                    values:
                    - "portworx"
                topologyKey: kubernetes.io/hostname
                namespaces:
                - "kube-system"
          hostNetwork: true
          containers:
          - name: nginx
            image: nginx
            ports:
            - containerPort: 80
            volumeMounts:
            - name: nginx-persistent-storage
              mountPath: /usr/share/nginx/html
          volumes:
          - name: nginx-persistent-storage
            persistentVolumeClaim:
              claimName: px-nginx-pvc
      ```

It can also be configured on cluster level by adding `spec.stork.args.webhook-controller` Set to `true` in StorageCluster to make Stork the default scheduler for workloads using Portworx volumes:

```text
apiVersion: core.libopenstorage.org/v1
kind: StorageCluster
metadata:
  name: portworx
  namespace: kube-system
spec:
  stork:
    enabled: true
    args:
      webhook-controller: true
```

## Uncordon the node

You can now uncordon the node using: `kubectl uncordon <node>`

If you want to permanently decommision the node, you can skip the following step.

## (Optional) Rejoin node to the cluster

If you want Portworx to start again on this node and join as a new node, follow the [node rejoin steps](/operations/operate-kubernetes/k8s-node-rejoin).

## (FlashArray cloud drives only) Disconnect or delete drives from your FlashArray

If you are using Pure FlashArray as a cloud drive provider, follow the additional instructions to [decommission FlashArray nodes](/operations/operate-kubernetes/uninstall/decomission-fa/).