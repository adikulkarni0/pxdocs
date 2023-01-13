---
title: "Configure migrations to use service accounts"
keywords: cloud, backup, restore, snapshot, DR, migration, kubemotion
description: Configure migrations to use service accounts
hidden: true
---

If you set up migrations and migration schedules using user accounts, you will encounter token expiration-related errors. To avoid these errors, Portworx, Inc. recommends setting up migration and migration schedules using service accounts.

In contrast to user accounts, which expire after a specified interval of time has passed, service account tokens do not expire. Using service accounts ensures that you will not encounter token expiration-related errors. See the [User accounts versus service accounts](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#user-accounts-versus-service-accounts) section of the Kubernetes documentation for more details about the differences between service accounts and user accounts.

Perform the following steps on the **destination cluster** to configure migrations to use service accounts.

## Create a service account and a cluster role binding

1. Create a file called `service-account-migration.yaml` with the following content, specifying the `namespace:` to match one of the existing namespaces in your cluster. For this example we will use the `default` namespace: 

    ```text
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: migration
      namespace: default
    ```

2. Apply the spec:

    ```text
    kubectl apply -f service-account-migration.yaml
    ```
    {{<info>}}
  **NOTE:** If you are using Kubernetes version 1.24 or newer, you also need to create a secret. In the example below, the name in the annotation `kubernetes.io/service-account.name` must match the name of the service account that you created.

```text
   apiVersion: v1
   kind: Secret
   metadata:
     name: migration
     namespace: default
     annotations: 
       kubernetes.io/service-account.name: migration
   type: kubernetes.io/service-account-token
```
  Apply the secret:

```text
kubectl apply -f <migrationsecretname>.yaml
```
    {{</info>}}

3. Create a file called `cluster-role-binding-migration.yaml` with the following content, specifying the `namespace:` field to match the namespace in the previous step:

    ```text
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: migration-clusterrolebinding
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: migration
      namespace: default
    ```
    {{<info>}}**NOTE:** The `roleRef.name` field is set to `cluster-admin`. For details about super-user access, see the [User-facing roles](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles) section of the Kubernetes documentation.{{</info>}}
  
4. Apply the spec:

    ```text
    kubectl apply -f cluster-role-binding-migration.yaml
    ```

## Create a kubeconfig file

1. Download the {{< direct-download url="/samples/async-dr-script/create-migration-config.sh" name="create-migration-config.sh" >}} script file. Edit the file and change the values of the `SERVER` and `NAMESPACE` variables to match your environment.

2. To create a kubeconfig file, enter the following commands:

    ```text
    chmod +x create-migration-config.sh && ./create-migration-config.sh > ~/.kube/migration-config.conf
    ```

3. Set the value of the `KUBECONFIG` environment variable to point to the kubeconfig file that you created in the previous step:

    ```text
    export KUBECONFIG=~/.kube/migration-config.conf
    ```

## Generate a cluster pair

1. To generate a cluster pair using this service account, enter the following command:

    ```text
    storkctl generate clusterpair -n <migrationnamespace> <remotecluster> --kubeconfig ~/.kube/migration-config.conf  > clusterpair.yaml
    ```
    
    * `<remotecluster>`: The Kubernetes object that will be created on the source cluster representing the pair relationship.
    * `<migrationnamespace>`: The Kubernetes namespace for the source cluster that you want to migrate to the destination cluster.

2. Copy the `clusterpair.yaml` file to your source cluster, modify the `options` section to match your environment, and apply it. Depending on whether you want to configure Portworx for asynchronous or synchronous disaster recovery, follow the steps in one of the following pages:

   * [Asynchronous disaster recovery](/operations/operate-kubernetes/disaster-recovery/async-dr/generate-apply-clusterpair/#generate-a-clusterpair-spec-on-the-destination-cluster)
   * [Synchronous disaster recovery](/operations/operate-kubernetes/disaster-recovery/px-metro/2-pair-clusters/#generate-a-clusterpair-on-the-destination-cluster)

