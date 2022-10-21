---
title: Import existing buckets using the Portworx Object Service
linkTitle: Import existing buckets
weight: 600
keywords: buckets, pbc, pba, bucket, s3, import
description: This guide is a step-by-step tutorial on how to import existing buckets with Portworx.
series: k8s-buckets
---

{{<info>}}
**NOTE:** Portworx Object Service is an early access feature.
{{</info>}}

This page describes how to import existing buckets using either [AWS S3](#aws-s3) or [Pure FlashBlade](#pure-flashblade).

## AWS S3

Use the following steps to get started with pre-provisioned buckets:

1. Create a new file named `pxbucketclass.yaml`, replacing `region` and `object.portworx.io/endpoint` with your desired AWS S3 region and endpoint:

    ```text
    apiVersion: object.portworx.io/v1alpha1
    kind: PXBucketClass
    metadata:
      name: pbclass-s3
    region: us-west-1
    deletionPolicy: Delete
    parameters:
      object.portworx.io/backend-type: S3Driver
      object.portworx.io/endpoint: s3.us-west-1.amazonaws.com
    ```

2. Create the PXBucketClass object:

    ```text
    kubectl apply -f pxbucketclass.yaml
    ```

3. Create a new file named `pxbucketaccess.yaml`:

    ```text
    apiVersion: object.portworx.io/v1alpha1
    kind: PXBucketAccess
    metadata:
      name: s3-pba-existing
      namespace: default
    spec:
      bucketClassName: pbclass-s3
      existingBucketId: bucket-name-aws
    ```

4. Once the bucket access is granted, its `ACCESSGRANTED` state will be marked as `true` in the CustomResource:

    ```text
    kubectl get pxbucketaccess
    ```
    ```output
    NAME              ACCESSGRANTED   CREDENTIALSSECRETNAME               BUCKETID               BACKENDTYPE
    s3-pba-existing   true            px-os-credentials-s3-pba-existing   existing-bucket-id     S3Driver
    ```

6. A secret `px-os-credentials-s3-pba-existing` will be created with all necessary bucket info:

    ```text
    kubectl get secret px-os-credentials-s3-pba -o yaml
    ```
    ```output
    apiVersion: v1
    data:
      access-key-id: <ACCESS-KEY-ID>
      bucket-id: <BUCKET-ID>
      endpoint: <ENDPOINT>
      region: <REGION>
      secret-access-key: <SECRET-ACCESS-KEY>
    kind: Secret
    metadata:
      creationTimestamp: "2022-08-03T21:27:25Z"
      finalizers:
      - finalizers.object.portworx.io/access-secret
      name: px-os-credentials-s3-pba
      namespace: default
      resourceVersion: "16022682"
      uid: 49aecbbd-c911-48cf-95ea-9e9d30aba97c
    type: Opaque
    ```

## Pure FlashBlade

Use the following steps to get started with pre-provisioned buckets:

1. Create a new file named `pxbucketclass.yaml`, replacing `region` and `object.portworx.io/endpoint` with your desired Pure FlashBlade region and endpoint:

    ```text
    apiVersion: object.portworx.io/v1alpha1
    kind: PXBucketClass
    metadata:
      name: pbclass-fb
    deletionPolicy: Delete
    parameters:
      object.portworx.io/backend-type: PureFBDriver
      object.portworx.io/endpoint: fb.fb.endpoint.example.com
    ```

2. Create the PXBucketClass object:

    ```text
    kubectl apply -f pxbucketclass.yaml
    ```

3. Create a new file named `pxbucketaccess.yaml`:

    ```text
    apiVersion: object.portworx.io/v1alpha1
    kind: PXBucketAccess
    metadata:
      name: fb-pba-existing
      namespace: default
    spec:
      bucketClassName: pbclass-fb
      existingBucketId: bucket-name-aws
    ```

4. Once the bucket access is granted, its `ACCESSGRANTED` state will be marked as `true` in the CustomResource:

    ```text
    kubectl get pxbucketaccess
    ```
    ```output
    NAME              ACCESSGRANTED   CREDENTIALSSECRETNAME               BUCKETID               BACKENDTYPE
    fb-pba-existing   true            px-os-credentials-fb-pba-existing   existing-bucket-id     PureFBDriver
    ```

5. A secret `px-os-credentials-fb-pba-existing` is created with all necessary bucket info:

    ```text
    kubectl get secret px-os-credentials-fb-pba -o yaml
    ```
    ```output
    apiVersion: v1
    data:
      access-key-id: <ACCESS-KEY-ID>
      bucket-id: <BUCKET-ID>
      endpoint: <ENDPOINT>
      region: <REGION>
      secret-access-key: <SECRET-ACCESS-KEY>
    kind: Secret
    metadata:
      creationTimestamp: "2022-08-03T21:27:25Z"
      finalizers:
      - finalizers.object.portworx.io/access-secret
      name: px-os-credentials-fb-pba
      namespace: default
      resourceVersion: "16022682"
      uid: 49aecbbd-c911-48cf-95ea-9e9d30aba97c
    type: Opaque
    ```

## Related topics

* [Portworx Object Service Reference](/reference/crd/object-service/)