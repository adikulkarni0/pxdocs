---
title: Scale an Elasticsearch cluster with Portworx on Kubernetes
linkTitle: Scale Elasticsearch
keywords: install, elasticsearch, kibana, kubernetes, k8s, scaling, failover
description: Find out how to easily scale Elasticsearch on Kubernetes using Portworx to preserve state!
weight: 400
noicon: true
---
The following instructions explain how to scale an Elasticsearch cluster with Portworx on Kubernetes.

Portworx runs as a Disaggregated or Converged Deployment architectures model. When you add a new node to your Kubernetes cluster, Portworx  will run on the node with the model you chose. Refer to [Deployment architectures for Portworx](/cloud-references/deployment-arch/) for more details.

1. Scale your Elasticsearch Cluster. Edit the Elasticsearch object and update `spec.nodeSets.count` to `4`:

   ```text
   kubectl -n elastic-system edit Elasticsearch
   ```
   ```output
   elasticsearch.elasticsearch.k8s.elastic.co/elasticsearch edited
   ```

2. List the associated pods to see the newly created Elasticsearch instance:

    ```text
    watch kubectl -n elastic-system get pods --selector='elasticsearch.k8s.elastic.co/cluster-name=elasticsearch'
    ```

    ```output
    NAME                      READY   STATUS    RESTARTS   AGE
    elasticsearch-es-node-0   1/1     Running   0          67m
    elasticsearch-es-node-1   1/1     Running   0          67m
    elasticsearch-es-node-2   1/1     Running   0          67m
    elasticsearch-es-node-3   1/1     Running   0          3m43s
    ```
3. List Elasticsearch cluster nodes:

    ```text
    kubectl exec elasticsearch-es-node-0 -n elastic-system -- curl -u "elastic:$ESPASSWORD"  -k "https://elasticsearch-es-http:9200/_cat/nodes?v"
    ```

    ```output
    ip              heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
    192.168.143.10            43          82   0    1.64    2.03     2.04 cdfhilmrstw -      elasticsearch-es-node-2
    192.168.148.113           57          77   7    2.73    3.55     2.83 cdfhilmrstw -      elasticsearch-es-node-3
    192.168.147.138           26          84   1    1.20    1.55     1.72 cdfhilmrstw -      elasticsearch-es-node-0
    192.168.45.31             27          85   1    1.30    2.56     2.65 cdfhilmrstw *      elasticsearch-es-node-1
    ```
4. List Elasticsearch PVCs:

    ```text
    kubectl -n elastic-system get pvc
    ```

    ```output
    NAME                                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    elasticsearch-data-elasticsearch-es-node-0   Bound    pvc-fa12f4d1-f2a6-4a99-83a1-c7a3e5555f41   50Gi       RWO            px-csi-db      69m
    elasticsearch-data-elasticsearch-es-node-1   Bound    pvc-c76c17d7-4600-4407-a0dc-d1e920903be2   50Gi       RWO            px-csi-db      69m
    elasticsearch-data-elasticsearch-es-node-2   Bound    pvc-8a7e27db-6a7e-4377-a936-7f678ca8ba07   50Gi       RWO            px-csi-db      69m
    elasticsearch-data-elasticsearch-es-node-3   Bound    pvc-1283ecde-fa04-4d84-9a8b-8452579d08d4   50Gi       RWO            px-csi-db      5m52s
    ```

5. List Elasticsearch Portworx volumes:

    ```text
    kubectl exec $PX_POD -n kube-system -- /opt/pwx/bin/pxctl volume list
    ```

    ```output
    ID                      NAME                                            SIZE    HA      SHARED  ENCRYPTED       PROXY-VOLUME    IO_PRIORITY     STATUS                          SNAP-ENABLED
    876851488987433344      pvc-1283ecde-fa04-4d84-9a8b-8452579d08d4        50 GiB  3       no      no              no              LOW             up - attached on 10.13.25.216   no
    12772348667588675       pvc-8a7e27db-6a7e-4377-a936-7f678ca8ba07        50 GiB  3       no      no              no              LOW             up - attached on 10.13.25.240   no
    1013728801369127826     pvc-c76c17d7-4600-4407-a0dc-d1e920903be2        50 GiB  3       no      no              no              LOW             up - attached on 10.13.25.29    no
    695731562422595940      pvc-fa12f4d1-f2a6-4a99-83a1-c7a3e5555f41        50 GiB  3       no      no              no              LOW             up - attached on 10.13.25.229   no
    ```