---
title: Object storage operations
weight: 100
keywords: create buckets, dynamic provisioning of buckets, kubernetes, k8s,
description: Learn how to create and use buckets for object storage
series: k8s-object-svc
hidesections: true
---

{{<info>}}
**NOTE:** Portworx Object Service is an early access feature.
{{</info>}}

## Overview

Portworx Object Service allows storage admins to provision object storage buckets with various backing providers using custom Kubernetes objects. You control this provisioning with a set of `CustomResourceDefinition` entries that are managed by Stork, which are as follows:

* [PXBucketClass](/reference/crd/object-service/#pxbucketclass)
* [PXBucketClaim](/reference/crd/object-service/#pxbucketclaim)
* [PXBucketAccess](/reference/crd/object-service/#pxbucketaccess)

## Supported object services

Portworx Object Service provides an abstraction to use any object backend through Kubernetes. The following object backends are supported during early access:

* [Pure FlashBlade](https://www.purestorage.com/products/file-and-object/flashblade.html)
* [AWS S3](https://aws.amazon.com/s3/)

Proceed to one of the following sections about Portworx Object Service operations.

{{<homelist series="k8s-buckets">}}
