---
title: Access via Non-Root Users
weight: 600
keywords: access a portworx volume, non-root user, pre-provisioned volumes, PV, PVC, Kubernetes, k8s
description: Find out how to access a Portworx volume (PVC/PV) with a non-root user
aliases:
    - /portworx-install-with-kubernetes/storage-operations/create-pvcs/access-via-non-root-users/
---
This document describes how to access a Portworx Volume (PVC/PV) as a non-root user. In Kubernetes, by default all the Persistent Volumes are accessible only by the root user. However you can modify the application pod spec to allow a specific set of users to access the Persistent Volume as explained below.

#### Modify the application POD spec

You can modify the PodSecurityContext section of a Pod to include an `fsGroup` field. Here is a snippet of a pod spec file

```text
  spec:
    # Allow non-root user to access PersistentVolume
    securityContext:
      fsGroup: 1000
    containers:
```

The `fsGroup` field specifies that the given special supplemental group with ID 1000 is associated with all Containers in the Pod. When you specify a `fsGroup` kubernetes will change the ownership of the volume to be owned by the pod.

* The owning GID will be that of the fsGroup.
* The setgid bit is set so that new files created in the volume are owned by the GID.
* The permission bits are OR’d with rw-rw—- For more information refer to the kubernetes [docs](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
