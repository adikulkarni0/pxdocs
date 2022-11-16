---
title: Failover Elasticsearch with Portworx on Kubernetes
linkTitle: Failover Elasticsearch
keywords: install, elasticsearch, kibana, kubernetes, k8s, scaling, failover
description: Find out how to easily deploy Elasticsearch and Kibana on Kubernetes using Portworx to preserve state!
weight: 500
noicon: true
---
The following instructions explain how to failover an Elasticsearch cluster with Portworx on Kubernetes.

## Pod Failover

Portworx provides durable storage for the Elasticsearch pods.

If an Elasticsearch pod fails, Kubernetes will schedule a new pod on another node. The StatefulSet with Portworx as the volume reattaches to this new pod. 

Perform the following instructions to simulate a failure scenario by cordoning a node so that Kubernetes does not schedule new pods on it, and delete a pod manually to simulate a failure scenario.

1. List all Elasticsearch Pod with the scheduled nodes:
    ```text
    kubectl -n elastic-system get pods --selector='elasticsearch.k8s.elastic.co/cluster-name=elasticsearch' -o wide
    ```

    ```output
    NAME                      READY   STATUS    RESTARTS   AGE   IP                NODE                  NOMINATED NODE   READINESS GATES
    elasticsearch-es-node-0   1/1     Running   0          75m   192.168.147.138   portworx-demo-node0   <none>           <none>
    elasticsearch-es-node-1   1/1     Running   0          75m   192.168.45.31     portworx-demo-node4   <none>           <none>
    elasticsearch-es-node-2   1/1     Running   0          75m   192.168.143.10    portworx-demo-node1   <none>           <none>
    elasticsearch-es-node-3   1/1     Running   0          11m   192.168.148.113   portworx-demo-node2   <none>           <none>
    ```

2. Cordon one of the nodes, in the case `portworx-demo-node2`:
    ```text
    kubectl cordon portworx-demo-node2
    ```

    ```output
    node/portworx-demo-node2 cordoned
    ```

3. Get all nodes:
    ```text
    kubectl get nodes
    ```

    ```output
    NAME                   STATUS                     ROLES                  AGE     VERSION
    portworx-demo-master   Ready                      control-plane,master   6h20m   v1.23.13
    portworx-demo-node0    Ready                      <none>                 6h20m   v1.23.13
    portworx-demo-node1    Ready                      <none>                 6h20m   v1.23.13
    portworx-demo-node2    Ready,SchedulingDisabled   <none>                 6h20m   v1.23.13
    portworx-demo-node3    Ready                      <none>                 6h20m   v1.23.13
    portworx-demo-node4    Ready                      <none>                 6h20m   v1.23.13
    ```

4. Find the Portworx volumes that are attached to the respective Elasticsearch pods. The following command returns the association between the Pods and the PVCs, showing the namespace, pod name, and the PVC:

    ```text
    kubectl get po -o json -n elastic-system | jq -j '.items[] | "\(.metadata.namespace), \(.metadata.name), \(.spec.volumes[].persistentVolumeClaim.claimName)\n"' | grep -v null
    ```

    ```output
    elastic-system, elasticsearch-es-node-0, elasticsearch-data-elasticsearch-es-node-0
    elastic-system, elasticsearch-es-node-1, elasticsearch-data-elasticsearch-es-node-1
    elastic-system, elasticsearch-es-node-2, elasticsearch-data-elasticsearch-es-node-2
    elastic-system, elasticsearch-es-node-3, elasticsearch-data-elasticsearch-es-node-3
    ```

5. Simulate a failure scenario by deleting the pod running on node `portworx-demo-node2`:
    ```text
    kubectl -n elastic-system delete po/elasticsearch-es-node-3 
    ```

    ```output
    pod "elasticsearch-es-node-3" deleted
    ```

6. Watch the pods:
    ```text
    kubectl -n elastic-system get pods --selector='elasticsearch.k8s.elastic.co/cluster-name=elasticsearch' -o wide
    ```

    ```output
    NAME                      READY   STATUS    RESTARTS   AGE   IP                NODE                  NOMINATED NODE   READINESS GATES
    elasticsearch-es-node-0   1/1     Running   0          85m   192.168.147.138   portworx-demo-node0   <none>           <none>
    elasticsearch-es-node-1   1/1     Running   0          85m   192.168.45.31     portworx-demo-node4   <none>           <none>
    elasticsearch-es-node-2   1/1     Running   0          85m   192.168.143.10    portworx-demo-node1   <none>           <none>
    elasticsearch-es-node-3   1/1     Running   0          3m    192.168.143.11    portworx-demo-node1   <none>           <none>
    ```

7. Verify that the same volume has been attached back to the pod which was scheduled post failover.

    ```text
    kubectl get po -o json -n elastic-system | jq -j '.items[] | "\(.metadata.namespace), \(.metadata.name), \(.spec.volumes[].persistentVolumeClaim.claimName)\n"' | grep -v null
    ```

    ```output
    elastic-system, elasticsearch-es-node-0, elasticsearch-data-elasticsearch-es-node-0
    elastic-system, elasticsearch-es-node-1, elasticsearch-data-elasticsearch-es-node-1
    elastic-system, elasticsearch-es-node-2, elasticsearch-data-elasticsearch-es-node-2
    elastic-system, elasticsearch-es-node-3, elasticsearch-data-elasticsearch-es-node-3
    ```

## Node Failover

A StatefulSet can have an unreachable node for various reasons, such as:

- The node is down for maintenance.
- The node is down unexpectedly.
- There has been a network partition.

Kubernetes has no way to determine the cause, so it will not schedule the StatefulSet, and the pods running on those nodes will enter the `Terminating` or `Unknown` state after a timeout. If there's a network partition, when the partition heals, Kubernetes will complete the deletion of the Pod and remove it from the API server. It will then schedule a new pod to honor the replication requirements mentioned in the Podspec. For further information, see [StatefulSet Pod Deletion](https://kubernetes.io/docs/tasks/run-application/force-delete-stateful-set-pod/) in the Kubernetes documentation.

Decomissioning a Kubernetes node deletes the node object form the APIServer.
Before that, you need to decomission your Portworx node from the cluster.
To do this, follow the steps mentioned in [Decommision a Portworx node](/operations/operate-kubernetes/uninstall/decommission-a-node).
Once done, delete the Kubernetes node if you want to delete it permanently.
