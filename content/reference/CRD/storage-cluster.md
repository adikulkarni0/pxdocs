---
title: StorageCluster
description: Explanation of Portworx Enterprise Operator and fields in StorageCluster object
keywords: portworx, operator, storagecluster
weight: 100
---

The Portworx cluster configuration is specified by a Kubernetes CRD (CustomResourceDefinition) called StorageCluster. The StorageCluster object acts as the definition of the Portworx Cluster.

The `StorageCluster` object provides a Kubernetes native experience. You can manage your Portworx cluster just like any other application running on Kubernetes. That is, if you create or edit the `StorageCluster` object, the operator will create or edit the Portworx cluster in the background.

To generate a `StorageCluster` spec customized for your environment, point your browser to  [PX-Central](https://central.portworx.com), and click "Install and Run" to start the Portworx spec generator. Then, the wizard will walk you through all the necessary steps to create a `StorageCluster` spec customized for your environment.

Note that using the Portworx spec generator is the recommended way of generating a `StorageCluster` spec. However, if you want to generate the `StorageCluster` spec manually, you can refer to the [StorageCluster Examples](#storagecluster-examples) and [StorageCluster Schema](#storagecluster-schema) sections.

## StorageCluster Examples

This section provides a few examples of common Portworx configurations you can use for manually configuring your Portworx cluster. Update the default values in these files to match your environment.

* Portworx with internal KVDB, configured to use all unused devices on the system.

    ```text
    apiVersion: core.libopenstorage.org/v1
    kind: StorageCluster
    metadata:
      name: portworx
      namespace: kube-system
    spec:
      image: portworx/oci-monitor:{{% currentVersion %}}
      kvdb:
        internal: true
      storage:
        useAll: true
    ```

* Portworx with external ETCD and Stork as default scheduler.

    ```text
    apiVersion: core.libopenstorage.org/v1
    kind: StorageCluster
    metadata:
      name: portworx
      namespace: kube-system
    spec:
      image: portworx/oci-monitor:{{% currentVersion %}}
      kvdb:
        endpoints:
        - etcd:http://etcd-1.net:2379
        - etcd:http://etcd-2.net:2379
        - etcd:http://etcd-3.net:2379
        authSecret: px-kvdb-auth
      stork:
        enabled: true
        args:
          health-monitor-interval: "100"
          webhook-controller: "true"
    ```

* Portworx with Security enabled.

    ```text
    apiVersion: core.libopenstorage.org/v1
    kind: StorageCluster
    metadata:
      name: portworx
      namespace: kube-system
    spec:
      image: portworx/oci-monitor:{{% currentVersion %}}
      security:
        enabled: true
    ```

* Portworx with Security enabled, guest access disabled, a custom self signed issuer/secret location, and five day token lifetime.

    ```text
    apiVersion: core.libopenstorage.org/v1
    kind: StorageCluster
    metadata:
      name: portworx
      namespace: kube-system
    spec:
      image: portworx/oci-monitor:{{% currentVersion %}}
      security:
        enabled: true
        auth:
          guestAccess: 'Disabled'
          selfSigned:
            issuer: 'openstorage.io'
            sharedSecret: 'px-shared-secret'
            tokenLifetime: '5d'
    ```

* Portworx with update and delete strategies, and placement rules.

    {{<info>}}**NOTE:** From Kubernetes version 1.24 and newer, the label key `node-role.kubernetes.io/master` is replace by  `node-role.kubernetes.io/control-plane`.{{</info>}}

    ```text
    apiVersion: core.libopenstorage.org/v1
    kind: StorageCluster
    metadata:
      name: portworx
      namespace: kube-system
    spec:
      image: portworx/oci-monitor:{{% currentVersion %}}
      updateStrategy:
        type: RollingUpdate
        rollingUpdate:
          maxUnavailable: 20%
      deleteStrategy:
        type: UninstallAndWipe
      placement:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: px/enabled
                operator: NotIn
                values:
                - "false"
              - key: node-role.kubernetes.io/control-plane
                operator: DoesNotExist
              - key: node-role.kubernetes.io/worker
                operator: Exists
        tolerations:
        - key: infra/node
          operator: Equal
          value: "true"
          effect: NoExecute
    ```

* Portworx with custom image registry, network interfaces, and miscellaneous options.

    ```text
    apiVersion: core.libopenstorage.org/v1
    kind: StorageCluster
    metadata:
      name: portworx
      namespace: kube-system
    spec:
      image: portworx/oci-monitor:{{% currentVersion %}}
      imagePullPolicy: Always
      imagePullSecret: regsecret
      customImageRegistry: docker.private.io/repo
      network:
        dataInterface: eth1
        mgmtInterface: eth2
      secretsProvider: vault
      runtimeOptions:
        num_io_threads: "10"
      env:
      - name: VAULT_ADDRESS
        value: "http://10.0.0.1:8200"
    ```

* Portworx with node specific overrides. Use different devices or no devices on different set of nodes.

    ```text
    apiVersion: core.libopenstorage.org/v1
    kind: StorageCluster
    metadata:
      name: portworx
      namespace: kube-system
    spec:
      image: portworx/oci-monitor:{{% currentVersion %}}
      storage:
        devices:
        - /dev/sda
        - /dev/sdb
      nodes:
      - selector:
          labelSelector:
            matchLabels:
              px/storage: "nvme"
        storage:
          devices:
          - /dev/nvme1
          - /dev/nvme2
      - selector:
          labelSelector:
            matchLabels:
              px/storage: "false"
        storage:
          devices: []
    ```

* Portworx with a cluster domain defined.

  ```text
      apiVersion: core.libopenstorage.org/v1
      kind: StorageCluster
      metadata:
        name: portworx
        namespace: kube-system
      annotations:
        portworx.io/misc-args: "-cluster_domain example-cluster-domain-name”
  ```

## StorageCluster Schema

This section explains the fields used to configure the `StorageCluster` object.

| Field | Description |Type| Default |
| --- | --- | --- | --- |
| spec.<br> image| Specifies the Portworx monitor image. | `string` | None |
| spec.<br> imagePullPolicy | Specifies the image pull policy for all the images deployed by the operator. It can take one of the following values: `Always` or `IfNotPresent` | `string` | `Always` |
| spec.<br>imagePullSecret | If Portworx pulls images from a secure repository, you can use this field to pass it the name of the secret. Note that the secret should be in the same namespace as the `StorageCluster` object. | `string` | None |
| spec.<br>customImageRegistry | The custom container registry server Portworx uses to fetch the Docker images. You may include the repository as well. | `string` | None |
| spec.<br>secretsProvider | The name of the secrets provider Portworx uses to store your credentials. To use features like cloud snapshots or volume encryption, you must configure a secret store provider. Refer to the [Secret store management page](/operations/key-management/) for more details. | `string` |  None |
| spec.<br>runtimeOptions | A collection of key-value pairs that overwrites the runtime options. | `map[string]string` | None |
| spec.<br>security | An object for specifying PX-Security configurations. Refer to the [Operator Security page](/operations/operate-kubernetes/authorization/enable/) for more details. | `object` | None |
| spec.<br>featureGates | A collection of key-value pairs specifying which Portworx features should be enabled or disabled. [^1] | `map[string]string` | None |
| spec.<br>env[] | A list of [Kubernetes like environment variables](https://github.com/kubernetes/api/blob/master/core/v1/types.go#L1826). Similar to how environment variables are provided in Kubernetes, you can directly provide values to Portworx or import them from a source like a `Secret`, `ConfigMap`, etc. | `[]object` | None |
| spec.<br>metadata.<br>annotations | A map of components and custom annotations. [^2] | map[string]map[string]string | None |
| spec.<br>metadata.<br>labels | A map of components and custom labels. [^3] | map[string]map[string]string | None |
| spec.<br>resources.<br>requests.<br>cpu | Specifies the cpu that the Portworx container requests; for example: `"4000m"` |  `string`  | None |
| spec.<br>resources.<br>requests.<br>memory | Specifies the memory that the Portworx container requests; for example: `"4Gi"` | `string` | None |

[^1]: As an example, here's how you can enable the `CSI` feature.

    For Operator 1.8 and higher:

    ```text
    spec:
      csi:
        enabled: true
        installSnapshotController: false
    ```
    For Operator 1.7 and earlier:

    ```text
    spec:
      featureGates:
        CSI: "true"
    ```
    Please note that you can also use `CSI: "True"` or `CSI: "1"`.

[^2]: The following example configures custom annotations. Change `<custom-domain/custom-key>: <custom-val>` to whatever `key: val` pairs you wish to provide.

    ```text
    spec:
      metadata:
        annotations:
          pod/storage:
            <custom-domain/custom-key>: <custom-val>
          service/portworx-api:
            <custom-domain/custom-key>: <custom-val>
          service/portworx-service:
            <custom-domain/custom-key>: <custom-val>
          service/portworx-kvdb-service:
            <custom-domain/custom-key>: <custom-val>
    ```
    Note that `StorageCluster.spec.metadata.annotations` is different from `StorageCluster.metadata.annotations`.
    Currently, custom annotations are supported on following types of components:

    | Type | Components |
    | --- | --- |
    | Pod | storage pods |
    | Service | portworx-api<br> portworx-service<br> portworx-kvdb-service |

[^3]: The following example configures labels for the `portworx-api` service. Change `<custom-label-key>: <custom-val>` to whatever `key: val` pairs you wish to provide.

    ```text
    spec:
      metadata:
        labels:
          service/portworx-api:
            <custom-label-key>: <custom-val>
    ```
    Note that `StorageCluster.spec.metadata.labels` is different from `StorageCluster.metadata.labels`. Currently, custom labels are only supported on the `portworx-api` service.

### KVDB configuration

This section explains the fields used to configure Portworx with a KVDB. Note that, if you don't specify the endpoints, the operator starts Portworx with the internal KVDB.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>kvdb.<br>internal | Specifies if Portworx starts with the [internal KVDB](/concepts/internal-kvdb). | `boolean` | `true` |
| spec.<br>kvdb.endpoints[]<br> | A list of endpoints for your external key-value database like ETCD. This field takes precedence over the `spec.kvdb.internal` field. That is, if you specify the endpoints, Portworx ignores the `spec.kvdb.internal` field and it uses the external KVDB. | `[]string` | None |
| spec.<br>kvdb.<br>authSecret | Indicates the name of the secret Portworx uses to authenticate against your KVDB. The secret must be placed in the same namespace as the `StorageCluster` object. The secret should provide the following information: <br> - `username` (optional) <br> - `password` (optional) <br> - `kvdb-ca.crt` (the CA certificate) <br> - `kvdb.key` (certificate key) <br> - `kvdb.crt` (etcd certificate) <br> - `acl-token` (optional) <br>For example, create a directory called etcd-secrets, copy the files into it and create a secret with `kubectl -n kube-system create secret generic px-kvdb-auth --from-file=etcd-secrets/`| `string` | None |

### Storage configuration

This section provides details about the fields used to configure the storage for your Portworx cluster. If you don't specify a device, the operator sets the `spec.storage.useAll` field to `true`.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>storage.<br>useAll | If set to `true`, Portworx uses all available, unformatted, and unpartitioned devices. [^4] | `boolean` | `true` |
| spec.<br>storage.<br>useAllWithPartitions | If  set to `true`, Portworx uses all the available and unformatted devices. [^4] | `boolean` |  `false` |
| spec.<br>storage.<br>forceUseDisks | If set to `true`, Portworx uses a device even if there's a file system on it. Note that Portworx may wipe the drive before using it. | `boolean` | `false` |
| spec.<br>storage.<br>devices[] | Specifies the list of devices Portworx should use. | `[]string` | None |
| spec.<br>storage.<br>cacheDevices[] | Specifies the list of cache devices Portworx should use. | `[]string` | None |
| spec.<br>storage.<br>journalDevice | Specifies the device Portworx uses for journaling. | `string` | None |
| spec.<br>storage.<br>systemMetadataDevice | Indicates the device Portworx uses to store metadata. For better performance, specify a system metadata device when using Portworx with the internal KVDB. | `string` | None |
| spec.<br>storage.<br>kvdbDevice | Specifies the device Portworx uses to store internal KVDB data. | `string` | None |

[^4]: Note that Portworx ignores this filed if you specify the storage devices using the `spec.storage.devices` field.

### Cloud storage configuration

This section explains the fields used to configure Portworx with cloud storage. Once the cloud storage is configured, Portworx manages the cloud disks automatically based on the provided specs. Note that the `spec.storage` fields take precedence over the fields presented in this section. Make sure the `spec.storage` fields are empty when configuring Portworx with cloud storage.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>cloudStorage.<br>provider | Specifies the cloud provider name, such as: pure, azure, aws, gce, vsphere. | `string` | None |
| spec.<br>cloudStorage.<br>deviceSpecs[] | A list of the specs for your cloud storage devices. Portworx creates a cloud disk for every device. | `[]string` | None |
| spec.<br>cloudStorage.<br>journalDeviceSpec | Specifies the cloud device Portworx uses for journaling. | `string` | None |
| spec.<br>cloudStorage.<br>systemMetadataDeviceSpec | Indicates the cloud device Portworx uses for metadata. For performance, specify a system metadata device when using Portworx with the internal KVDB. | `string` | None |
| spec.<br>cloudStorage.<br>kvdbDeviceSpec | Specifies the cloud device Portworx uses for an internal KVDB. | `string` | None |
| spec.<br>cloudStorage.<br>maxStorageNodesPerZone | Indicates the maximum number of storage nodes per zone. If this number is reached, and a new node is added to the zone, Portworx doesn't provision drives for the new node. Instead, Portworx starts the node as a compute-only node. | `uint32` | None |
| spec.<br>cloudStorage.<br>maxStorageNodes | Specifies the maximum number of storage nodes. If this number is reached, and a new node is added, Portworx doesn't provision drives for the new node. Instead, Portworx starts the node as a compute-only node. As a best practice, it is recommended to use the `maxStorageNodesPerZone` field. | `uint32` | None |

### Network configuration

This section describes the fields used to configure the network settings. If these fields are not specified, Portworx auto-detects the network interfaces.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>network.<br>dataInterface | Specifies the network interface Portworx uses for data traffic. | `string` | None |
| spec.<br>network.<br>mgmtInterface| Indicates the network interface Portworx uses for control plane traffic. | `string` | None |

### Volume configuration

This section describes the fields used to configure custom volume mounts for Portworx pods.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>volumes[].<br>name | Unique name for the volume. | `string` | None |
| spec.<br>volumes[].<br>mountPath | Path within the Portworx container at which the volume should be mounted. Must not contain the ':' character. | `string` | None |
| spec.<br>volumes[].<br>mountPropagation | Determines how mounts are propagated from the host to container and the other way around. | `string` | None |
| spec.<br>volumes[].<br>readOnly | Volume is mounted read-only if true, read-write otherwise. | `boolean` | false |
| spec.<br>volumes[].<br>[secret\|configMap\|hostPath] | Specifies the location and type of the mounted volume. This is similar to the VolumeSource schema of a Kubernetes pod volume. | `object` | None |


### Placement rules

You can use the placement rules to specify where Portworx should be deployed. By default, the operator deploys Portworx on all worker nodes.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>placement.<br>nodeAffinity | Use this field to restrict Portwox on certain nodes. It works similarly to the [Kubernetes node affinity](https://github.com/kubernetes/api/blob/master/core/v1/types.go#L2692) feature. | `object` | None |
| spec.<br>placement.<br>tolerations[] | Specifies a list of tolerations that will be applied to Portworx pods so that they can run on nodes with matching taints.| `[]object` | None |

For Operator 1.8 and higher, if you have topology labels [`topology.kubernetes.io/region`](https://kubernetes.io/docs/reference/labels-annotations-taints/#topologykubernetesioregion) or [`topology.kubernetes.io/zone`](https://kubernetes.io/docs/reference/labels-annotations-taints/#topologykubernetesiozone) specified on worker nodes, Operator would deploy Stork, Stork scheduler, CSI and PVC controller pods with [`topologySpreadConstraints`](https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/) to distribute pod replicas across Kubernetes failure domains.

### Update strategy

This section provides details on how to specify an update strategy.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>updateStrategy.<br>type | Indicates the update strategy. Currently, Portworx supports the following update strategies- `RollingUpdate` and `OnDelete`. | `object` | `RollingUpdate` |
| spec.<br>updateStrategy.<br>rollingUpdate.<br>maxUnavailable | Similarly to how Kubernetes rolling update strategies work, this field specifies how many nodes can be down at any given time. Note that you can specify this as a number or percentage. | `int` or `string` | `1` |
| spec.<br>autoUpdateComponents | Indicates the update strategy for the component images (such as Stork, Autopilot, Prometheus, and so on). Portworx supports the following auto update strategies for the component images:<br> <ul><li>`None`: Updates the component images only when the Portworx image changes in `StorageCluster.spec.image`.</li><li>`Once`: Updates the component images once even if the Portworx image does not change. This is useful when the component images on the manifest server change due to bug fixes.</li><li>`Always`: Regularly checks for the updates on the manifest server, and updates the component images if required.</li>| `string` | None |

### Delete/Uninstall strategy

This section provides details on how to specify an uninstall strategy for your Portworx cluster.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>deleteStrategy.<br>type | Indicates what happens when the Portworx `StorageCluster` object is deleted. By default, there is no delete strategy, which means only the Kubernetes components deployed by the operator are removed.  The Portworx `systemd` service continues to run, and the Kubernetes applications using the Portworx volumes are not affected. Portworx supports the following delete strategies: <br> - `Uninstall` - Removes all Portworx components from the system and leaves the devices and KVDB intact. <br> - `UninstallAndWipe` - Removes all Portworx components from the system and wipes the devices and metadata from KVDB.  | `string`| None |

### Monitoring configuration

This section provides details on how to enable monitoring for Portworx.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>monitoring.<br>prometheus.<br>enabled | Enables or disables a Prometheus cluster. | `boolean` | `false` |
| spec.<br>monitoring.<br>prometheus.<br>exportMetrics | Expose the Portworx metrics to an external or operator deployed Prometheus. | `boolean` | `false` |
| spec.<br>monitoring.<br>prometheus.<br>alertManager.<br>enabled | Enable Prometheus Alertmanager | `boolean` |
| spec.<br>monitoring.<br>telemetry.<br>enabled | Enable telemetry and metrics collector | `boolean` | `false` |

### Stork configuration

This section describes the fields used to manage the Stork deployment through the Portworx operator.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>stork.<br>enabled | Enables or disables Stork at any given time. | `boolean` | `true` |
| spec.<br>stork.<br>image | Specifies the Stork image. | `string` | None |
| spec.<br>stork.<br>lockImage | Enables locking Stork to the given image. When set to false, the Portworx Operator will overwrite the Stork image to a recommended image for given Portworx version. | `boolean` | `false` |
| spec.<br>stork.<br>args | A collection of key-value pairs that overrides the default Stork arguments or adds new arguments. | `map[string]string` | None |
| spec.<br>stork.<br>args.admin-namespace | Set up a cluster's admin namespace for migration. <br>Refer to [admin namespace](/operations/operate-kubernetes/migration/cluster-admin-namespace/) for more information | `string` | `kube-system` |
| spec.<br>stork.<br>args.verbose | Set to `true` to enable verbose logging | `boolean` | `false` |
| spec.<br>stork.<br>args.webhook-controller | Set to `true` to make Stork the default scheduler for workloads using Portworx volumes | `boolean` | `false` |
| spec.<br>stork.<br>env[] | A list of [Kubernetes like environment variables](https://github.com/kubernetes/api/blob/master/core/v1/types.go#L1826) passed to Stork. | `[]object` | None |
| spec.<br>stork.<br>volumes[] | A list of volumes passed to Stork pods. The schema is similar to the [top-level volumes](#volume-configuration). | `[]object` | None |

### CSI configuration

This section provides details on how to configure CSI for the StorageCluster. Note this is for Operator 1.8 and higher only.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>csi.<br>enabled | Flag indicating whether CSI needs to be installed for the storage cluster. | `boolean` | true |
| spec.<br>csi.<br>installSnapshotController | Flag indicating whether CSI Snapshot Controller needs to be installed for the storage cluster. | `boolean` | false |

### Autopilot configuration

This section provides details on how to deploy and manage Autopilot.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>autopilot.<br>enabled | Enables or disables Autopilot at any given time. | `boolean` | `false` |
| spec.<br>autopilot.<br>image | Specifies the Autopilot image. | `string` | None |
| spec.<br>autopilot.<br>lockImage | Enables locking Autopilot to the given image. When set to false, the Portworx Operator will overwrite the Autopilot image to a recommended image for given Portworx version. | `boolean` | `false` |
| spec.<br>autopilot.<br>providers[] | List of data providers for Autopilot. | `[]object` | None |
| spec.<br>autopilot.<br>providers[].<br>name | Unique name of the data provider. | `string` | None |
| spec.<br>autopilot.<br>providers[].<br>type | Type of the data provider. For instance, `prometheus`. | `string` | None |
| spec.<br>autopilot.<br>providers[].<br>params | Map of key-value params for the provider. | `map[string]string` | None |
| spec.<br>autopilot.<br>args | A collection of key-value pairs that overrides the default Autopilot arguments or adds new arguments. | `map[string]string` | None |
| spec.<br>autopilot.<br>env[] | A list of [Kubernetes like environment variables](https://github.com/kubernetes/api/blob/master/core/v1/types.go#L1826) passed to Autopilot. | `[]object` | None |
| spec.<br>autopilot.<br>volumes[] | A list of volumes passed to Autopilot pods. The schema is similar to the [top-level volumes](#volume-configuration). | `[]object` | None |

### Node specific configuration

This section provides details on how to override certain cluster level configuration for individual or group of nodes.

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>nodes[] | A list of node specific configurations. | `[]object` | None |
| spec.<br>nodes[].<br>selector | Selector for the node(s) to which the configuration in this section will be applied. | `object` | None |
| spec.<br>nodes[].<br>selector.<br>.nodeName | Name of the node to which this configuration will be applied. Node name takes precedence over `selector.labelSelector`. | `string` | None |
| spec.<br>nodes[].<br>selector.<br>.labelSelector | [Kubernetes style label selector](https://github.com/kubernetes/apimachinery/blob/master/pkg/apis/meta/v1/types.go) for nodes to which this configuration will be applied. | `object` | None |
| spec.<br>nodes[].<br>network | Specify network configuration for the selected nodes, similar to the one [specified at cluster level](#network-configuration). If this network configuration is empty, then cluster level values are used. | `object` | None |
| spec.<br>nodes[].<br>storage | Specify storage configuration for the selected nodes, similar to the one [specified at cluster level](#storage-configuration). If some of the config is left empty, the cluster level storage values are passed to the nodes. If you don't want to use a cluster level value and set the field to empty, then explicitly set an empty value for it so no value is passed to the nodes. For instance, set `spec.nodes[0].storage.kvdbDevice: ""`, to prevent using the KVDB device for the selected nodes. | `object` | None |
| spec.<br>nodes[].<br>env | Specify extra environment variables for the selected nodes. Cluster level environment variables are combined with these and sent to the selected nodes. If same variable is present at cluster level, then the node level variable takes precedence. | `object` | None |
| spec.<br>nodes[].<br>runtimeOptions | Specify runtime options for the selected nodes. If specified, cluster level options are ignored and only these runtime options are passed to the nodes.  | `object` | None |

### Security Configuration

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.<br>security.<br>enabled | Enables or disables Security at any given time. | `boolean` | `false` |
| spec.<br>security.<br>auth.<br>guestAccess | Determines how the guest role will be updated in your cluster. The options are Enabled, Disabled, or Managed. Managed will cause the operator to ignore updating the system.guest role. Enabled and Disabled will allow or disable guest role access, respectively. | `string` | Enabled |
| spec.<br>security.<br>auth.<br>selfSigned.<br>tokenLifetime | The length operator-generated tokens will be alive until being refreshed. | `string` | 24h |
| spec.<br>security.<br>auth.<br>selfSigned.<br>issuer | The issuer name to be used when configuring PX-Security. This field maps to the `PORTWORX_AUTH_JWT_ISSUER` environment variable in the Portworx Daemonset. | `string` | operator.portworx.io |
| spec.<br>security.<br>auth.<br>selfSigned.<br>sharedSecret | The Kubernetes secret name for retrieving and storing your shared secret. This field can be used to add a pre-existing shared secret or for customizing which secret name the operator will use for its auto-generated shared secret key. This field maps to the `PORTWORX_AUTH_JWT_SHAREDSECRET` environment variable in the Portworx Daemonset. | `string` | px-shared-secret |

### StorageCluster Annotations

| Annotation | Description |
| --- | --- |
| `portworx.io/misc-args` | Arguments that you specify in this annotation are passed to portworx container verbatim. For example:<br> `portworx.io/misc-args: "-cluster_domain datacenter1 --tracefile-diskusage 5"` <br>Note that you cannot use `=` to specify the value of an argument. |
| `portworx.io/service-type` | Annotation to configure type of services created by operator. For example:<br>`portworx.io/service-type: "LoadBalancer"` to specify `LoadBalancer` type for all services.<br> For Operator 1.8.1 and higher, the value can be a list of service names and corresponding type configurations split in `;`, service not specified will use its default type. For example:<br>`portworx.io/service-type: "portworx-service:LoadBalancer;portworx-api:ClusterIP;portworx-kvdb-service:LoadBalancer"` |
| `portworx.io/portworx-proxy` | Portworx proxy is needed to allow Kubernetes to communicate with Portworx when using the in-tree driver.  The proxy is automatically enabled if you run Portworx in a namespace other than kube-system and not using the default 9001 start port. You can set the annotation to **false** to disable the proxy. For example: <br>`portworx.io/portworx-proxy: "false"`|