---
title: "Configure migrations to use service accounts"
keywords: cloud, backup, restore, snapshot, DR, migration, kubemotion
description: Configure migrations to use service accounts
hidden: true
---

If you set up migrations and migration schedules using user accounts, you will encounter token expiration-related errors. To avoid these errors, Portworx, Inc. recommends setting up migration and migration schedules using service accounts.

In contrast to user accounts that expire after a specified interval of time has passed, service account tokens do not expire. Using service accounts ensures that you will not encounter token expiration-related errors. See the [User accounts versus service accounts](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#user-accounts-versus-service-accounts) section of the Kubernetes documentation for more details about the differences between service accounts and user accounts.

<!-- It would be better if we could be more specific, meaning that we should provide the text of the error message-->

Perform the following steps on the **destination cluster** to configure migrations to use service accounts.

## Create a service account and a cluster role binding

1. Create a file called `service-account-migration.yaml` with the following content, adjusting the value of the `metadata.namespace` field to match the one you use to set up your migration:

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

3. Create a file called `cluster-role-binding-migration.yaml` with the following content, adjusting the `subjects.namespace` field to match with the namespace you wish to set up your migration:

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
    {{<info>}}
  **NOTE:** The `roleRef.name` field is set to `cluster-admin`. For details about super-user access, see the [User-facing roles](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles) section of the Kubernetes documentation. <!-- "This gives full control over all resources in the cluster and all namespaces". Is this considered a good practice?-->
{{</info>}}
  
4. Apply the spec:

    ```
    kubectl apply -f clusterrolebinding-migration.yaml
    ```

## Create a kubeconfig file

1. Create a file called `create-migration-config.sh` with the following content, adjusting the values of the `SERVER` and `NAMESPACE` variables to match your environment:

    ```text
    SERVICE_ACCOUNT=migration
    NAMESPACE=default
    SERVER=https://<SERVER-ADDRESS:PORT>

    SERVICE_ACCOUNT_TOKEN_NAME=$(kubectl -n ${NAMESPACE} get serviceaccount ${SERVICE_ACCOUNT} -o yaml | grep "name: $SERVICE_ACCOUNT-token" | cut -f 3 -d " ")
    SERVICE_ACCOUNT_TOKEN=$(kubectl -n ${NAMESPACE} get secret ${SERVICE_ACCOUNT_TOKEN_NAME} -o "jsonpath={.data.token}" | base64 --decode)
    SERVICE_ACCOUNT_CERTIFICATE=$(kubectl -n ${NAMESPACE} get secret ${SERVICE_ACCOUNT_TOKEN_NAME} -o "jsonpath={.data['ca\.crt']}")

    cat <<END
    apiVersion: v1
    kind: Config
    clusters:
    - name: default-cluster
      cluster:
        certificate-authority-data: ${SERVICE_ACCOUNT_CERTIFICATE}
        server: ${SERVER}
    contexts:
    - name: default-context
      context:
        cluster: default-cluster
        namespace: ${NAMESPACE}
        user: ${SERVICE_ACCOUNT}
    current-context: default-context
    users:
    - name: ${SERVICE_ACCOUNT}
      user:
        token: ${SERVICE_ACCOUNT_TOKEN}
    END
    ```

2. To create a kubeconfig file, enter the following commands:

    ```text
    chmod +x create-migration-config.sh && ./create-migration-config.sh > ~/.kube/migration-config.conf
    ```

3. Set the value of the `KUBECONFIG` environment variable to point to the kubeconfig file you created in the previous step:

    ```text
    export KUBECONFIG=~/.kube/migration-config.conf
    ```

## Generate a cluster pair

1. To generate a cluster pair using this service account, enter the following `storkctl generate clusterpair` command:

    ```text
    storkctl generate clusterpair mig-clusterpair --kubeconfig ~/.kube/migration-config.conf  > mig-clusterpair.yaml
    ```

2. Copy the `mig-clusterpair.yaml` file to your source cluster, modify the `options` section to match your environment and apply it. Depending on whether you want to configure Portworx for asynchronous or synchronous disaster recovery, follow the steps in one of the following pages:

  - [Asynchronous disaster recovery](https://docs.portworx.com/portworx-install-with-kubernetes/disaster-recovery/async-dr/#enable-disaster-recovery-mode)
  - [Synchronous disaster recovery](https://docs.portworx.com/portworx-install-with-kubernetes/disaster-recovery/px-metro/2-pair-clusters/#generate-a-clusterpair-on-the-destination-cluster)

