---
title: Install Portworx Object service
linkTitle: Install Object Service
weight: 100
keywords: buckets, pbc, pba, bucket, s3, install, setup
description: This guide is a step-by-step tutorial on how to install the Portworx Object Service.
series: k8s-buckets
---

{{<info>}}
**NOTE:** Portworx Object Service is an early access feature.
{{</info>}}

This page describes how to install Portworx Object Service.

## Prerequisites

To install Portworx Object Service, you must meet the following prerequisites:

* Use {{< pxEnterprise >}} 2.12.0 or newer
* Use Stork 2.12.0 or newer
* Provide access to an AWS S3 or Pure FlashBlade secret access key ID and secret access key
* Use a cluster running Kubernetes 1.17 or newer

## Installation

Portworx Object Service objects are managed by Stork, and they interact with a target {{< pxEnterprise >}} instance. The Portworx Object Service SDK is located in the target {{< pxEnterprise >}} instance. This allows you to create buckets, delete buckets, and provide or revoke access to buckets.

Additionally, you must provide access to the backend bucket service through environment variables. Because Portworx Object Service is in early access, extra steps are required to enable and set up the Portworx Object Service controller. The following steps allow {{< pxEnterprise >}} to create and provide access to buckets on behalf of the credentials provided:

1. Enable the Portworx Object Service controller flag in Stork by adding the following `args` to your StorageCluster spec:

    ```text
    spec:
      ...
      stork:
        enabled: true
        args:
          px-object-controller: "true"
    ```

2. Create a new Kubernetes secret with your AWS S3 or Pure FlashBlade access key ID and secret access key:

    * For AWS S3, add the following:
    
        ```text
        kubectl create secret generic px-object-s3-admin-credentials \ 
            --from-literal=access-key-id=ACCESS_KEY \ 
            --from-literal=secret-access-key=SECRET_ACCESS_KEY 
        ```
        
    * For Pure FlashBlade, add the following:
    
        ```text
        kubectl create secret generic px-object-fb-admin-credentials \ 
            --from-literal=access-key-id=ACCESS_KEY \ 
            --from-literal=secret-access-key=SECRET_ACCESS_KEY 
        ```

3. Add environment variables for bucket credentials to your StorageCluster spec.<br><br>

    * For AWS S3, add the following:

        ```text
        spec:
          env:
          - name: OBJECT_SERVICE_S3_ACCESS_KEY_ID
            valueFrom:
              secretKeyRef:
                name: px-object-s3-admin-credentials
                key: access-key-id
          - name: OBJECT_SERVICE_S3_SECRET_ACCESS_KEY
            valueFrom:
              secretKeyRef:
                name: px-object-s3-admin-credentials
                key: secret-access-key
        ```

        * `OBJECT_SERVICE_S3_ACCESS_KEY_ID`: An AWS S3 Access Key ID credential generated in the AWS Portal.
        * `OBJECT_SERVICE_S3_SECRET_ACCESS_KEY`: An AWS S3 Secret Access Key credential generated in the AWS Portal.

    * For Pure FlashBlade, add the following:

        ```text
        spec:
          env:
          - name: OBJECT_SERVICE_FB_ACCESS_KEY_ID
            valueFrom:
              secretKeyRef:
                name: px-object-fb-admin-credentials
                key: access-key-id
          - name: OBJECT_SERVICE_FB_SECRET_ACCESS_KEY
            valueFrom:
              secretKeyRef:
                name: px-object-fb-admin-credentials
                key: secret-access-key
        ```

        * `OBJECT_SERVICE_FB_ACCESS_KEY_ID`: A Pure FlashBlade Access Key ID credential provided by the FlashBlade admin.
        * `OBJECT_SERVICE_FB_SECRET_ACCESS_KEY`: A Pure FlashBlade Secret Access Key credential provided by the FlashBlade admin.

## Related topics

* [Portworx Object Service Reference](/reference/crd/object-service/)
