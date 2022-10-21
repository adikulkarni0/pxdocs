---
title: Portworx Object Service Reference
linkTitle: Object Service
weight: 900
keywords: buckets, pbc, pba, bucket, s3, install, reference
description: This details all of the different parameters and variables to use with the Portworx Object Service.
series: k8s-buckets
---

{{<info>}}
**NOTE:** Portworx Object Service is an early access feature.
{{</info>}}

## Environment Variables

Various settings are available for setting up the [Portworx Object Service](/operations/operate-kubernetes/storage-operations/object/) controller and further configuring how it operates. This reference page includes all possible configurations. 

### Portworx Operator

In the StorageCluster Stork spec, the following argument enables the Portworx Object Service:

```
spec:
  ...
  stork:
    enabled: true
    args:
      px-object-controller: "true"
```

### Portworx Enterprise

Set the following environment variables in the StorageCluster `spec.env` to customize the Portworx Object Service credentials:

* `OBJECT_SERVICE_S3_ACCESS_KEY_ID`: An AWS S3 Access Key ID credential generated in the AWS Portal.
* `OBJECT_SERVICE_S3_SECRET_ACCESS_KEY`: An AWS S3 Secret Access Key credential generated in the AWS Portal.
* `OBJECT_SERVICE_FB_ACCESS_KEY_ID`: A Pure FlashBlade Access Key ID credential provided by the FlashBlade admin.
* `OBJECT_SERVICE_FB_SECRET_ACCESS_KEY`: A Pure FlashBlade Secret Access Key credential provided by the FlashBlade admin.

### Stork

Set the following environment variables in the StorageCluster `spec.stork.env` to customize the Portworx Object Service controller:

* `WORKER_THREADS`: The number of worker threads to use in the Portworx Object Service Stork controller. Default is 4.
* `RETRY_INTERVAL_START`: Initial retry interval of failed bucket creation/access or deletion/revoke. It doubles with each failure, up to retry-interval-max. Default is 1 second.
* `RETRY_INTERVAL_MAX`: Maximum retry interval of failed bucket/access creation or deletion/revoke. Default is 5 minutes.

## CustomResourceDefinitions

The Portworx Object Service introduces multiple new Kubernetes objects to allow for easy provisioning and access to object backends:

### PXBucketClass 

The PXBucketClass serves as a template for creating new BucketClaim and BucketAccess objects. It contains the necessary metadata for interacting with different backends.

#### Example

```
apiVersion: object.portworx.io/v1alpha1
kind: PXBucketClass
metadata:
  name: <name>
region: <region>
deletionPolicy: [ Delete | Retain ]
parameters:
  object.portworx.io/backend-type: [ S3Driver | PureFBDriver ]
  object.portworx.io/endpoint: <S3-endpoint>
  object.portworx.io/clear-bucket: [ true | false ]
```

#### Schema

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| region | The region to be used for your backend object provider. | `string` | None |
| deletionPolicy | Indicates whether the bucket should be deleted in the backend object provider or not. Available options are only `Delete` or `Retain` | `string` | None |
| parameters\[object.portworx.io/backend-type\] | The backend object provider to use for this PXBucketClass. Supported options are `S3Driver` or `PureFBDriver`  | `string` | None |
| parameters\[object.portworx.io/endpoint\] | The endpoint to use for connecting to the backend object provider.  | `string` | Detected based on region |
| parameters\[object.portworx.io/clear-bucket\] | Indicates whether or not to delete all objects when a bucket is being deleted. If objects exist in a bucket and this is set to false, the deletion may fail. | `string` | "false" |


### PXBucketClaim

The PXBucketClaim object represents a new bucket in the backend object system.

#### Example

```
apiVersion: object.portworx.io/v1alpha1
kind: PXBucketClaim
metadata:
  name: <name>
  namespace: <namespace>
spec:
  bucketClassName: <bucket-class-name>
```

#### Schema

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.bucketClassName | The name of the PXBucketClass to use when provisioning a new bucket | `string` | None |


### PXBucketAccess

The PXBucketAccess object controls how access is provisioned to a PXBucketClaim or a pre-existing bucketId in the backend system.

#### Example
```
apiVersion: object.portworx.io/v1alpha1
kind: PXBucketAccess
metadata:
  name: <name>
  namespace: <namespace>
spec:
  bucketClassName:  <bucket-class-name>
  bucketClaimName:  <bucket-claim-name>
  existingBucketId: <pre-existing-bucket-id>
```

#### Schema

| Field | Description | Type | Default |
| --- | --- | --- | --- |
| spec.bucketClassName | The name of the PXBucketClass to use when provisioning a new bucket | `string` | None |
| spec.bucketClaimName | The name of the PXBucketClaim to reference when providing access to a new bucket | `string` | None |
| spec.existingBucketId | The optional bucket ID in the backend object provider for providing access to an existing bucket | `string` | None |
