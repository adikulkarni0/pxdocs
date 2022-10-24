---
title: ResourceTransformation
description: ResourceTransformation CRD reference
keywords: Portworx, operator, ResourceTransformation, CRD, reference
weight: 300
---

Metro and asynchronous disaster recovery (DR) involves migrating Kubernetes resources from a source cluster to a destination cluster. To ensure applications can come up correctly on the destination clusters, you may need to modify resources such as `Service`, `ServiceAccount` or `ConfigMap` to work as intended on your destination cluster. The ResourceTransformation feature allows you to define a set of rules that modify the Kubernetes resources before they are migrated to the destination cluster.

ResourceTransformation is a custom resource (CR) that accepts change rules that stork follows to modify resources from a source cluster before applying them onto a destination cluster.

## ResourceTransformation schema

* **Path**: the YAML path for a given Kubernetes resource you want to perform an operation on. Provide this in dot notation, e.g. `metadata.labels`.
* **Value**: what value you want to update the selected resource to. For example, if `path` is `spec.replicas`, you can specify a value of `4`.
* **Type**: the currently supported data type you want to update values to. Type can be one of the following:

    | Type | Value | Example |
    | ---- | ----- | ------- |
    | KeyPair | Comma separated key value pair for patching map type in resource specs | a:b,c:d,e:f,g:h <br/> \<key1:value1\>, \<key2:value2\>, … \<keyN:valueN\> |
    | List    | Comma separated array element list for patching resource specs | [a,b,c,d] <br/> <val1,val2,…,valN> |
    | Int     | Integer type value for updating resources | 0 |
    | String  | String type values | “new-val” |
    | Bool    | Boolean values for setting path value | True, False |
{{<info>}}
**NOTE:** Complex types such as Array of KeyPairs is not supported.
{{</info>}}
* **Operations** What operation you want to perform on the given `path` in an unstructured Kubernetes object:

  * **Add**: sets a new nested `path` in an unstructured object specification with `value`. If `path` already exists, this will replace the existing value.
  * **Modify**: updates a nested path in an unstructured resource object with `value`. If the value is a `KeyPair` or `List` type, this will append `value` to the existing path.
  * **Delete**: removes the specified nested spec path from an unstructured object.

{{<info>}}
**NOTE:** You can specify multiple paths for the same resource and multiple resources in single ResourceTransformation CR.
{{</info>}}

## ResourceTransformation spec

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind : ResourceTransformation
metadata:
  name: name-of-transform-rules
specs:
 transformSpecs:
 - paths:
     - path: <spec-path-of-k8s-resources-to-modify>
       value: <value-to-be-updated-at-above-path>
       type: <type-of-value->
       operation: <operation-to-perform>
     - path: <spec-path-of-k8s-resources-to-modify>
       value: <value-to-be-updated-at-above-path>
       type: <type-of-value->
       operation: <operation-to-perform>
   resource: <k8s-resource-version/kind-format>
```

## Supported Resources

Portworx can currently perform transformations on the following resources:

* `ConfigMap`
* `Service`
* `Secret`
* `ServiceAccount`
* `Role`
* `RoleBinding`
* `ClusterRole`
* `ClusterRoleBinding`
* `Ingress`
* `NetworkPolicy`
* `Endpoint`


## Example 

The example in this topic modifies all Kubernetes services in the `mysql` namespace on the destination cluster during the migration based on the migration schedule.

### Define the source service object

This `Service` spec defines the source service object. Note the following:

* There's currently no `metadata.annotations` defined in the following service object. Secondly the `type` of service being used is `ClusterIP`.
* `metadata.labels` contains only the `app: mysql` key value pair.

```
apiVersion: v1
kind: Service
metadata:
 name: mysql-service
 labels:
   app: mysql
spec:
  type: ClusterIP
 selector:
   app: mysql
 ports:
   - name: transport
     port: 3306
```

### Create a ResourceTransformation spec

This `ResourceTransformation` spec does two things when it's called:

* Changes the type from `service` to `LoadBalancer`.
* Updates `metadata.labels`, adding the `handler: project` key value pair to it and replacing `app: mysql` with `app:mysql-2`.
* Adds new `metadata.annotations` to the service object.

```
apiVersion: stork.libopenstorage.org/v1alpha1
kind : ResourceTransformation
metadata:
  name: mysql-service-transform
  namespace: mysql
specs:
 transformSpecs:
 - paths:
     - path: "spec.type"
       value: "LoadBalancer"
       type: "string"
       operation: "modify"
     - path: "metadata.labels"
       value: "handler:project,app:mysql-2"
       type: "keypair"
       operation: "modify"
     - path: "metadata.annotations"
       value: "handler:project,app:mysql-2"
       type: "keypair"
       operation: "add"
   resource: "/v1/Service"
```

When migrated, Portworx will use the rules defined in this ResourceTransformation spec to modify the object it creates on the destination cluster.

### Use the ResourceTransformation Spec

A `MigrationSchedule` references the `ResourceTransformation` defined above in the `spec.template.spec.transformSpecs` field. When it triggers, the `MigrationSchedule` uses the `ResourceTransformation` defined above to change the object it deploys on the destination cluster:

```
apiVersion: stork.libopenstorage.org/v1alpha1
kind: MigrationSchedule
metadata:
 name: mysql-migration-transform-interval
 Namespace: mysql
spec:
 schedulePolicyName: migrate-every-5m
 template:
   spec:
     # This should be the name of the cluster pair
     clusterPair: remoteclusterpair
     # If set to false this will migrate only the volumes. No PVCs, apps, etc will be migrated
     includeResources: true
     # If set to false, the deployments and stateful set replicas will be set to
     # 0 on the destination. There will be an annotation with
     # "stork.openstorage.org/migrationReplicas" to store the replica count from the source
     startApplications: false
     # Update service resource as per transformation specs
     transformSpecs:
     - mysql-service-transform
     namespaces:
     - mysql-1-pvc-mysql-migration-transform-interval
```

