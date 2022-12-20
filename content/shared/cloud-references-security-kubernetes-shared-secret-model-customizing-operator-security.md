---
title: "Customize Security in Portworx"
keywords: authorization, portworx, security, rbac, jwt
hidden: true
---

This document guides you through optionally customizing your Portworx Operator Security configuration further to fit specific needs.

## Prerequisites

* Portworx Operator 1.4 or later
* PX-Security enabled

## Disable guest role access

Starting with Portworx 2.6.0 and later, the [system guest role](/concepts/authorization/overview#guest-access) is enabled by default. To turn off this feature, you can disable it in the StorageCluster spec:

{{<info>}}
**NOTE:** Once the guest role is disabled, volumes created without a token will only be accessible _with_ a token.
{{</info>}}

```text
apiVersion: core.libopenstorage.org/v1
kind: StorageCluster
metadata:
  name: px-cluster
  namespace: kube-system
spec:
  security:
    enabled: true
    auth:
      guestAccess: 'Disabled'
```

## Managing the guest role yourself

You can exercise finer control over the `system.guest` role by setting it to `managed` mode. This instructs the Operator to stop updating the [system guest role](/concepts/authorization/overview#guest-access), allowing you to [customize it yourself](/reference/cli/role/#re-enabling-the-system-guest-role).

To enter `managed` mode, set the value of the `spec.security.auth.guestAccess` field to `managed`:

```text
apiVersion: core.libopenstorage.org/v1
kind: StorageCluster
metadata:
  name: px-cluster
  namespace: kube-system
spec:
  security:
    enabled: true
    auth:
      guestAccess: 'Managed'
```

## Changing token lifetime

By default, the token is valid for 24 hours. You can optionally specify a different JWT token lifetime. The Operator then generates a token with that token lifetime and refreshes it for the user accordingly. 

```text
apiVersion: core.libopenstorage.org/v1
kind: StorageCluster
metadata:
  name: px-cluster
  namespace: kube-system
spec:
  security:
    enabled: true
    auth:
      tokenLifetime: '4h'
```

## Add a custom issuer, shared secret, and tokenLifetime to your StorageCluster

Add your `issuer`, `tokenLifetime`, and `sharedSecret` Kubernetes secret's name to the `spec.security.auth.selfSigned` object in your StorageCluster:

```text
apiVersion: core.libopenstorage.org/v1
kind: StorageCluster
metadata:
  name: px-cluster
  namespace: kube-system
spec:
  security:
    enabled: true
    auth:
      selfSigned:
        issuer: "portworx.com"
        sharedSecret: "px-shared-secret"
        tokenLifetime: "1h"
```
