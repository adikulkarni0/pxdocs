---
title: VolumePlacementStrategy CRD reference
keywords: portworx, storage class, container, Kubernetes, storage, Docker, k8s, flexvol, pv, persistent disk,StatefulSets, volume placement
description: Reference material for the VolumePlacementStrategy CRD.
aliases:
    - /portworx-install-with-kubernetes/storage-operations/create-pvcs/volume-placement-strategies/crd-reference/
---
Portworx provides a [CustomResouceDefinition (CRD)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions) called `VolumePlacementStrategy`. The specification for this CRD is composed of 4 main sections:

- [replicaAffinity](#replicaaffinity)
    - [Schema](#schema)
    - [Example use cases](#example-use-cases-of-replicaaffinity)
- [replicaAntiAffinity](#replicaantiaffinity)
    - [Schema](#schema-1)
    - [Example use cases](#example-use-cases)
- [volumeAffinity](#volumeaffinity)
    - [Schema](#schema-2)
    - [Example use cases](#example-use-cases-1)
- [volumeAntiAffinity](#volumeantiaffinity)
    - [Schema](#schema-3)
    - [Example use cases](#example-use-cases-2)

Additional Information:

- [topologyKey](#topologykey)
- [Templates](#templates)


### replicaAffinity

##### Schema

| Field  	| Description    | Optional? | Default |
|---------|----------------|-----------|---------|
|**enforcement**|Specifies if the given rule is required (hard) or preferred (soft). Portworx will fail volume creation if it cannot provision volumes matching a rule with hard enforcement enabled. |Yes|required|
|**matchExpressions**|matchExpressions is a list of label selector requirements. The requirements are ANDed.<br/><br/>The labels provided here are matched against the following:<ul><li>Kubernetes node labels</li><li>PVC labels</li><li>Portworx storage pool labels</li></ul>|Yes * <br/><br/>* required if topologyKey is empty | empty|
|**topologyKey** | Key for the node label that the system uses to denote a topology domain. The key can be for any node label that is present on the Kubernetes node. <br/><br/>{{<info>}}**NOTE:** Using topologyKey requires nodes to be consistently labelled, i.e. every node in the cluster must have an appropriate label matching topologyKey. If some or all nodes are missing the specified topologyKey label, it can lead to unintended behavior.{{</info>}}|Yes* <br/><br/>* required if matchExpressions is empty|empty|
|**weight**|Specifies the weight of the rule. If more than one rule matches, then the rule with the highest weight is applied.|Yes|0|
|**affected_replicas**|Specifies the number of volume replicas affected by this rule.|Yes|0 (Interpreted as all volume replicas)|

##### Example use cases of replicaAffinity

* {{< open-in-new-tab url="/samples/k8s/volume-placement-ssd-pool-affinity.yaml" name="Use SSD pools" >}}
* {{< open-in-new-tab url="/samples/k8s/volume-placement-ssd-sata-pool-spread-affinity.yaml" name="Use SSD and SATA and spread replicas evenly" >}}
* {{< open-in-new-tab url="/samples/k8s/volume-placement-one-ssd-other-sata-pool.yaml" name="First replica SSD, others SATA" >}}
* {{< open-in-new-tab url="/samples/k8s/volume-infra-node-affinity.yaml" name="Place volume on specific nodes , example infra etc" >}}
* {{< open-in-new-tab url="/samples/k8s/volume-rack-1-2-3-affinity.yaml" name="Place volume only on racks 1, 2 and 3" >}}


By default, Portworx automatically adds the following labels to each of its storage pools. These can be used for replica affinity rules to target replicas on a specific storage pool type:

* `iopriority`
* `medium (Media Type)`

### replicaAntiAffinity


##### Schema

| Field  	| Description    | Optional? | Default |
|---------|----------------|-----------|---------|
|**enforcement**|Specifies if the given rule is required (hard) or preferred (soft). Portworx will fail volume creation if it cannot provision volumes matching a rule with hard enforcement enabled. |Yes|required|
|**topologyKey**|Key for the node label that the system uses to denote a topology domain. The key can be for any node label that is present on the Kubernetes node. Using topologyKey requires nodes to be consistently labelled, i.e. every node in the cluster must have an appropriate label matching topologyKey. If some or all nodes are missing the specified topologyKey label, it can lead to unintended behavior.|Yes* * required if matchExpressions is empty|empty|

##### Example use cases

* {{< open-in-new-tab url="/samples/k8s/antiAffinityDataRacks.yaml" name="Antiaffinity to data pools and spread across racks" >}}
* {{< open-in-new-tab url="/samples/k8s/volume-placement-sata-pool-anti-affinity.yaml" name="Antiaffinity to not use SATA pools" >}}

### volumeAffinity



##### Schema

| Field  	| Description    | Optional? | Default |
|---------|----------------|-----------|---------|
|**enforcement**|Specifies if the given rule is required (hard) or preferred (soft) Portworx will fail volume creation if it cannot provision volumes matching a rule with hard enforcement enabled.|Yes|required|
|**topologyKey**|Key for the node label that the system uses to denote a topology domain. The key can be for any node label that is present on the Kubernetes node. <br/>Using topologyKey requires nodes to be consistently labelled, i.e. every node in the cluster must have an appropriate label matching topologyKey. If some or all nodes are missing the specified topologyKey label, it can lead to unintended behavior.|Yes|empty|
|**matchExpressions**|matchExpressions is a list of label selector requirements. The requirements are ANDed.<br/><br/>The labels provided here are matched against the following:<ul><li>Kubernetes node labels</li><li>PVC labels</li><li>Portworx</li> storage pool labels</li>|No|empty|
|**weight**|Specifies the weight of the rule. If more than one rule matches, then the rule with the highest weight is applied.|Yes|0|

##### Example use cases

* {{< open-in-new-tab url="/samples/k8s/volume-placement-postgres-volume-affinity.yaml" name="Collocate postgres volumes" >}}
* {{< open-in-new-tab url="/samples/k8s/volume-db-nodes-or-postgres-affinity.yaml" name="Place volume on DB type nodes or collocate with postgres volumes" >}}

### volumeAntiAffinity



##### Schema

| Field  	| Description    | Optional? | Default |
|---------|----------------|-----------|---------|
|**enforcement**|Specifies if the given rule is required (hard) or preferred (soft). Portworx will fail volume creation if it cannot provision volumes matching a rule with hard enforcement enabled.|Yes|required|
|**topologyKey**|Key for the node label that the system uses to denote a topology domain. The key can be for any node label that is present on the Kubernetes node. <br/>Using topologyKey requires nodes to be consistently labelled, i.e. every node in the cluster must have an appropriate label matching topologyKey. If some or all nodes are missing the specified topologyKey label, it can lead to unintended behavior.|Yes|empty|
|**matchExpressions**|matchExpressions is a list of label selector requirements. The requirements are ANDed.<br/><br/>The labels provided here are matched against the following:<ul><li>Kubernetes node labels</li><li>PVC labels</li><li>Portworx</li> storage pool labels</li>|No|empty|

##### Example use cases

* {{< open-in-new-tab url="/samples/k8s/volume-placement-cassandra-volume-anti-affinity.yaml" name="Do not collocate with other cassandra volumes" >}}

### topologyKey
If `topologyKey` is not set, by default the rule will avoid/colocate volumes at the node level using built-in unique node identifiers.

##### Portworx Built-in
`topologyKey` can use the built-in key `portworx.io/pool` that is set automatically as a unique key/value on every Portworx pool. This can be used with volumeAffinity/volumeAntiAffinity to control volume placement at the pool level to work for individual pools.


### Templates

Parameter values in `matchExpressions` may use predefined templates. The `VolumePlacementStrategy` will automatically resolve these templates at provision time as detailed below:

* key: `namespace`
    * `${pvc.namespace}`
        * Replaced with namespace of the `PersistentVolumeClaim` that triggered provisioning
        * Example: {{< open-in-new-tab url="/samples/k8s/volume-anti-affinity-namespace-template.yaml" name="Do not collocate with other app volumes within the same namespace" >}}
* key: `px/statefulset-pod`
    * `${pvc.statefulset-pod}`
        * Replaced with the `StatefulSet` pod name the `PersistentVolumeClaim` belongs to
        * Matches corresponding label key/value applied to the volume
        * Use this with multiple `volumeClaimTemplates` in a `StatefulSet` to relate volumes for the same replica pod to each other
        * Works with `Immediate` binding mode
        * Example: {{< open-in-new-tab url="/samples/k8s/statefulset-template-volume-affinity.yaml" name="Collocate volumes that belong to the same StatefulSet pod" >}}
