---
title: Create buckets using the Portworx Object Service
linkTitle: Create buckets
weight: 500
keywords: buckets, pbc, pba, bucket, fb, s3
description: This guide is a step-by-step tutorial on how to provision buckets with Portworx.
series: k8s-buckets
---

{{<info>}}
**NOTE:** Portworx Object Service is an early access feature.
{{</info>}}

This page describes how to create and provide access to a Portworx Bucket Claim using either [AWS S3](#aws-s3) or [Pure FlashBlade](#pure-flashblade).

## AWS S3

Use the following steps to get started with dynamically provisioned buckets.

### Provision a new bucket

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

3. Create a new file named `pxbucketclaim.yaml`:

    ```text
    apiVersion: object.portworx.io/v1alpha1
    kind: PXBucketClaim
    metadata:
      name: s3-pbc
      namespace: default
    spec:
      bucketClassName: pbclass-s3
    ```

4. Create the PXBucketClaim object:

    ```text
    kubectl apply -f pxbucketclaim.yaml
    ```

5. Once the bucket is provisioned, its `PROVISIONED` state will be listed as `true` in the `CustomResource`:

    ```text
    kubectl get pxbucketclaim
    ```
    ```output
    NAME     PROVISIONED   BUCKETID                                     BACKENDTYPE
    s3-pbc   true          px-os-06663fb0-d1bb-4b8a-914c-ac6595c2b721   S3Driver
    ```

### Provide Access to the PXBucketClaim

1. Create a new file named `pxbucketaccess.yaml`:

    ```text
    apiVersion: object.portworx.io/v1alpha1
    kind: PXBucketAccess
    metadata:
      name: s3-pba
      namespace: default
    spec:
      bucketClassName: pbclass-s3
      bucketClaimName: s3-pbc
    ```

2. Once the bucket access is granted, its `ACCESSGRANTED` state will be marked as `true` in the CustomResource:

    ```text
    kubectl get pxbucketaccess
    ```
    ```output
    NAME     ACCESSGRANTED   CREDENTIALSSECRETNAME      BUCKETID                                     BACKENDTYPE
    s3-pba   true            px-os-credentials-s3-pba   px-os-06663fb0-d1bb-4b8a-914c-ac6595c2b721   S3Driver
    ```

6. A secret `px-os-credentials-s3-pba` is created with all nessesary bucket info:

    ```text
    kubectl get secret px-os-credentials-s3-pba -o yaml
    ```
    ```output
    apiVersion: v1
    data:
      access-key-id: <access-key-id>
      bucket-id: <bucket-id>
      endpoint: <endpoint>
      region: <region>
      secret-access-key: <secret-access-key>
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

Use the following steps to get started with dynamically provisioned buckets.

### Provision a new bucket

1. Create a new file named `pxbucketclass.yaml`, removing `region` and replacing `object.portworx.io/endpoint` with your desired Pure FlashBlade endpoint:

    ```text
    apiVersion: object.portworx.io/v1alpha1
    kind: PXBucketClass
    metadata:
      name: pbclass-fb
    deletionPolicy: Delete
    parameters:
      object.portworx.io/backend-type: PureFBDriver
      object.portworx.io/endpoint: fb.s3.example.com
    ```

2. Create the PXBucketClass object:

    ```text
    kubectl apply -f pxbucketclass.yaml
    ```

3. Create a new file named `pxbucketclaim.yaml`:

    ```text
    apiVersion: object.portworx.io/v1alpha1
    kind: PXBucketClaim
    metadata:
      name: fb-pbc
      namespace: default
    spec:
      bucketClassName: pbclass-fb
    ```

4. Create the PXBucketClaim object:

    ```text
    kubectl apply -f pxbucketclaim.yaml
    ```

5. Once the bucket is provisioned, its `PROVISIONED` state will be listed as `true` in the CustomResource:

    ```text
    kubectl get pxbucketclaim
    ```
    ```output
    NAME     PROVISIONED   BUCKETID                                     BACKENDTYPE
    fb-pbc   true          px-os-06663fb0-d1bb-4b8a-914c-ac6595c2b721   PureFBDriver
    ```

### Provide access to the PXBucketClaim

1. Create a new file named `pxbucketaccess.yaml`:

    ```
    apiVersion: object.portworx.io/v1alpha1
    kind: PXBucketAccess
    metadata:
      name: fb-pba
      namespace: default
    spec:
      bucketClassName: pbclass-fb
      bucketClaimName: fb-pbc
    ```

2. Once the bucket access is granted, its `ACCESSGRANTED` state will be marked as `true` in the CustomResource:

    ```text
    kubectl get pxbucketaccess
    ```
    ```output
    NAME     ACCESSGRANTED   CREDENTIALSSECRETNAME      BUCKETID                                     BACKENDTYPE
    fb-pba   true            px-os-credentials-fb-pba   px-os-06663fb0-d1bb-4b8a-914c-ac6595c2b721   PureFBDriver
    ```

6. A secret `px-os-credentials-fb-pba` is created with all nessesary bucket info:

    ```text
    kubectl get secret px-os-credentials-fb-pba -o yaml
    ```
    ```output
    apiVersion: v1
    data:
      access-key-id: <access-key-id>
      bucket-id: <bucket-id>
      endpoint: <endpoint>
      secret-access-key: <secret-access-key>
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