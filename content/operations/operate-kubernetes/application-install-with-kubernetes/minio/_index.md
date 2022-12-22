---
title: MinIO on Kubernetes on Portworx
linkTitle: MinIO
keywords: install, Minio, object storage, kubernetes, k8s, scaling, failover
description: Find out how to easily deploy Minio on Kubernetes using Portworx to preserve state!
weight: 300
noicon: true
aliases:
    - /portworx-install-with-kubernetes/application-install-with-kubernetes/minio
---

## Overview
MinIO is a high-performance distributed object store with an S3 compatible API.
   
The following steps deploy the latest version of the MinIO operator and tenant on Kubernetes. Refer to the [MinIO documentation](https://min.io/docs/minio/kubernetes/upstream/operations/installation.html) for more information.

## Prerequisites

- Have a Kubernetes cluster running version 1.19 or newer.
- Have Portworx installed on your Kubernetes cluster. For details, see [Install Portworx](/install-portworx/).
- Meet the [MinIO prerequisites](https://min.io/docs/minio/kubernetes/upstream/operations/installation.html#prerequisites).

## Deploy the MinIO Operator

Follow the steps in the MinIO documentation to [deploy the MinIO Operator](https://min.io/docs/minio/kubernetes/upstream/operations/installation.html#procedure).

## Create a MinIO Tenant using the command line

{{<info>}}
**NOTE:** The example deployment in this document is not recommended for production environments. Refer to the [MinIO documentation](https://min.io/docs/minio/kubernetes/upstream/operations/install-deploy-manage/deploy-minio-tenant.html#minio-k8s-deploy-minio-tenant) for more details.
{{</info>}}

There are multiple ways to deploy MinIO tenants. In this example, we will create a tenant using the command line.

Portworx offers a set of Storage classes out of the box. In the example, we are using the **px-csi-db** StorageClass to provision Portworx volumes.

1. Check the storage classes and make sure that **px-csi-db** is set as the default:

    ```text
    kubectl get sc
    ```
    ```output
    px-csi-db (default)                  pxd.portworx.com                Delete          Immediate           true                   52m
    px-csi-db-cloud-snapshot             pxd.portworx.com                Delete          Immediate           true                   52m
    px-csi-db-cloud-snapshot-encrypted   pxd.portworx.com                Delete          Immediate           true                   52m
    ...
    px-db                                kubernetes.io/portworx-volume   Delete          Immediate           true                   52m
    px-db-cloud-snapshot                 kubernetes.io/portworx-volume   Delete          Immediate           true                   52m
    px-db-cloud-snapshot-encrypted       kubernetes.io/portworx-volume   Delete          Immediate           true                   52m
    ...
    px-replicated                        kubernetes.io/portworx-volume   Delete          Immediate           true                   52m
    px-replicated-encrypted              kubernetes.io/portworx-volume   Delete          Immediate           true                   52m
    stork-snapshot-sc                    stork-snapshot                  Delete          Immediate           true                   52m
    ```

    If you have a different default storage class, mark it as non-default and mark the **px-csi-db**  StorageClass as default. Refer to the [Kubernetes documentation](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/#changing-the-default-storageclass) for information about marking non-default and default storage classes.

2. Create a MinIO tenant namespace:

    ```text
    kubectl create ns minio-demo
    ```
    ```output
    namespace/minio-demo created
    ```

3. Create a tenant:

    ```text
    kubectl minio tenant create                    \
                        minio-demo                 \
                        --capacity 100Gi           \
                        --servers 1                \
                        --volumes 1                \
                        --namespace minio-demo     \
                        --storage-class px-csi-db
    ```
    ```output
    Tenant 'minio-demo' created in 'minio-demo' Namespace

      Username: <tenant-id>
      Password: <tenant-password>
      Note: Copy the credentials to a secure location. MinIO will not display these again.

    APPLICATION     SERVICE NAME            NAMESPACE       SERVICE TYPE    SERVICE PORT
    MinIO           minio                   minio-demo      ClusterIP       443
    Console         minio-demo-console      minio-demo      ClusterIP       9443
    ```

    In this example, the required settings are as follows:<br><br>

    * `TENANT_NAME`: The name of the tenant. In this example, we are using `minio-demo` as the tenant name.
    * `--capacity`: The total Raw size of storage across all volumes
    * `--servers`: The total number of Server Pods in the Tenant
    * `--volumes`: The total number of storage Volumes.
    * `--namespace`: The namespace required by the MinIO Tenant.
    * `--storage-class`: The storage class to use to provision Portworx Volumes.

4. Copy the credentials displayed in the output to a secure location.<br><br>

    {{<info>}}
**NOTE:** When creating the tenant, the Operator displays the access credentials to use for the tenant. This is the only time the credentials display.
    {{</info>}}

5. Check the tenant pod status: <!-- what is the user checking for? -->

    ```text
    kubectl get po -n minio-demo
    ```
    ```output
    NAME                                         READY   STATUS    RESTARTS       AGE
    minio-demo-log-0                             1/1     Running   0              2m20s
    minio-demo-log-search-api-6954696f58-rpr2n   1/1     Running   3 (119s ago)   2m20s
    minio-demo-prometheus-0                      2/2     Running   0              80s
    minio-demo-ss-0-0                            1/1     Running   0              2m20s
    ```

6. Check Portworx volumes: <!-- what is the user checking for? -->

    ```text
    kubectl get pvc -n minio-demo
    ```
    ```output
    NAME                                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    0-minio-demo-ss-0-0                             Bound    pvc-7f7fa6f2-ac70-42a1-b516-d456ddf8af1e   100Gi      RWO            px-csi-db      3m32s
    minio-demo-log-minio-demo-log-0                 Bound    pvc-40c6d9e7-0ea6-46b3-88c9-af8b15bd9084   5Gi        RWO            px-csi-db      3m32s
    minio-demo-prometheus-minio-demo-prometheus-0   Bound    pvc-db17d274-6b15-49b2-a72d-0336eb868566   5Gi        RWO            px-csi-db      2m32s
    ```

## Access the tenant

The Operator creates multiple services for the tenant.

1. To get service details, run the following command:

    ```text
    kubectl get svc -n minio-demo
    ```
    ```output
    minio                          ClusterIP   10.108.83.135    <none>        443/TCP    6m29s
    minio-demo-console             ClusterIP   10.110.166.206   <none>        9443/TCP   6m29s
    minio-demo-hl                  ClusterIP   None             <none>        9000/TCP   6m29s
    minio-demo-log-hl-svc          ClusterIP   None             <none>        5432/TCP   5m28s
    minio-demo-log-search-api      ClusterIP   10.96.72.119     <none>        8080/TCP   5m28s
    minio-demo-prometheus-hl-svc   ClusterIP   None             <none>        9090/TCP   4m28s
    ```

    {{<info>}}
**NOTE:**

* Applications should use the `minio` service for performing operations against the MinIO tenant.

* Administrators should use the `<TENANT-NAME>-console` service for accessing the MinIO Console and performing administrative operations.
    {{</info>}}

2. Use port-forwarding to expose each service and access the MinIO tenant for applications or the console UI for administration.<br><br>

    To port-forward the MinIO tenant, run the following command:

    ```text
    kubectl port-forward service/minio 8443:443 -n minio-demo
    ```
    ```output
    Forwarding from 127.0.0.1:8443 -> 9000
    Forwarding from [::1]:8443 -> 9000
    ```

    To port-forward the MinIO console, run the following command:
    
    ```text
    kubectl port-forward service/minio-demo-console  -n minio-demo 9443:9443
    ```
    ```output
    Forwarding from 127.0.0.1:9443 -> 9443
    Forwarding from [::1]:9443 -> 9443
    ```

## Access the MinIO tenant console and create a bucket

1. To temporarily connect to the MinIO tenant console, navigate to `https://localhost:9443`.

2. Use the credentials obtained during deployment and log in to the console.

3. Click **Create Bucket +** to start bucket creation:

    ![Minio Tenant Console UI](/img/MinioOperatortenantConsole.png)

4. Click **Create Bucket** to finish bucket creation:

    ![Minio Tenant Console UI2](/img/MinioOperatortenantBucket.png)

    The tenant console UI displays the list of buckets, including the bucket that you created:

    ![Minio Tenant Console UI3](/img/MinioOperatortenantBucket2.png)

## Application access to the MinIO tenant

Applications should use the `minio` service to perform operations against the MinIO tenant.

Use the credentials obtained during deployment for application access.

