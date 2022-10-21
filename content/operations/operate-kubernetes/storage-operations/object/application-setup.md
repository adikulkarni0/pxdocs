---
title: Application setup with bucket access
weight: 700
keywords: buckets, pbc, pba, bucket, s3, application
description: This guide is a step-by-step tutorial on how to use buckets with your applications.
series: k8s-buckets
---

{{<info>}}
**NOTE:** Portworx Object Service is an early access feature.
{{</info>}}

This page describes how to use a PXBucketAccess object with your application. Note that the following steps are the same for both Pure Flashblade and AWS S3 buckets.

1. In your application's `deployment.yaml` file, add all of the environment variables for your buckets as Kubernetes deployment secret references. For example:

    ```text
        env:
        - name: S3_ACCESS_KEY
        valueFrom:
            secretKeyRef:
                name: px-os-credentials-s3-pba
                key: access-key-id
        - name: S3_SECRET_KEY
        valueFrom:
            secretKeyRef:
                name: px-os-credentials-s3-pba
                key: secret-access-key
        - name: S3_BUCKET_NAME
        valueFrom:
            secretKeyRef:
                name: px-os-credentials-s3-pba
                key: bucket-id
        - name: S3_ENDPOINT
        valueFrom:
            secretKeyRef:
                name: px-os-credentials-s3-pba
                key: endpoint
        - name: S3_REGION
        valueFrom:
            secretKeyRef:
                name: px-os-credentials-s3-pba
                key: region
    ```

2. Apply the updates to your `deployment.yaml`:

    ```text
    kubectl apply -f deployment.yaml
    ```

## Related topics

* [Portworx Object Service Reference](/reference/crd/object-service/)