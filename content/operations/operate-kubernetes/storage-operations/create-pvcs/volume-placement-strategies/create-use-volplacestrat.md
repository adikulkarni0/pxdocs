---
title: Create and use VolumePlacementStrategies
keywords: portworx, storage class, container, Kubernetes, storage, Docker, k8s, flexvol, pv, persistent disk,StatefulSets, volume placement
description: Instructions for creating and using VolumePlacementStrategies.
aliases:
    - /portworx-install-with-kubernetes/storage-operations/create-pvcs/volume-placement-strategies/create-use-volplacestrat/
---
Create your VolumePlacementStrategy along with your other storage resources:

### Prerequisites

* **Portworx version**: 2.1.2 and above

### Construct a VolumePlacementStrategy spec

1. Create a YAML file containing the following common fields. All VolumePlacementStrategy CRDs use these fields:

      * `apiVersion` as `portworx.io/v1beta2`
      * `kind` as `VolumePlacementStrategy`
      * `metadata.name` with the name of your strategy

      Add any of the following affinity or antiaffinity sections to the spec:

      * [replicaAffinity](/operations/operate-kubernetes/storage-operations/create-pvcs/volume-placement-strategies/crd-reference#replicaaffinity)
      * [replicaAntiAffinity](/operations/operate-kubernetes/storage-operations/create-pvcs/volume-placement-strategies/crd-reference#replicaantiaffinity)
      * [volumeAffinity](/operations/operate-kubernetes/storage-operations/create-pvcs/volume-placement-strategies/crd-reference#volumeaffinity)
      * [volumeAntiAffinity](/operations/operate-kubernetes/storage-operations/create-pvcs/volume-placement-strategies/crd-reference#volumeantiaffinity)

      This example adds a volumeAffinity rule to colocate Postgres volumes for performance:

      ```text
      apiVersion: portworx.io/v1beta2
      kind: VolumePlacementStrategy
      metadata:
        name: postgres-volume-affinity
      spec:
        volumeAffinity:
          - matchExpressions:
            - key: app
              operator: In
              values:
                - postgres
      ```

3. Save and apply your spec with the `kubectl apply` command:

      ```text
      kubectl apply -f yourVolumePlacementStrategy.yaml
      ```

### Create other storage specs

#### Use a StorageClass

You can associate your VolumePlacementStrategy with a StorageClass and then reference that StorageClass in your PVC. 

1. Create a StorageClass that references the VolumePlacementStrategy you created in the **Construct a VolumePlacementStrategy spec** steps above by specifying the `placement_strategy` parameter with the name of your VolumePlacementStrategy:

      ```text
      apiVersion: storage.k8s.io/v1
      kind: StorageClass
      metadata:
        name: postgres-storage-class
      provisioner: kubernetes.io/portworx-volume
      parameters:
        placement_strategy: "postgres-volume-affinity"
      ```
2. Save and apply your StorageClass with the `kubectl apply` command:

      ```text
      kubectl apply -f yourVolumePlacementStrategy.yaml
      ```
3. Create a PVC which references the StorageClass you created above, specifying the `storageClass`

      ```text
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
         name: postgres-pvc
      spec:
         storageClassName: postgres-storage-class
         accessModes:
           - ReadWriteOnce
         resources:
           requests:
             storage: 2Gi
      ```
4. Save and apply your PVC with the `kubectl apply` command:

      ```text
      kubectl apply -f yourPVC.yaml
      ```

#### Reference a VolumePlacementStrategy directly in a PVC

You can reference your VolumePlacementStrategy directly in your PVC using an annotation.

1. Create a PVC which references the VolumePlacementStrategy you created in the **Construct a VolumePlacementStrategy spec** steps above by specifying `placement_strategy` as an annotation with the name of your VolumePlacementStrategy:

      ```text
      kind: PersistentVolumeClaim
        apiVersion: v1
        metadata:
          annotations: 
            placement_strategy: "postgres-volume-affinity"
          name: postgres-pvc
        spec:
          storageClassName: postgres-storage-class
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 2Gi
      ```

4. Save and apply your PVC with the `kubectl apply` command:

      ```text
      kubectl apply -f yourPVC.yaml
      ```

Once you've applied your volumePlacementStrategy, StorageClass, and PVC, Portworx deploys volumes according to the rules you defined. Portworx also follows VolumePlacementStrategies when it restores volumes.