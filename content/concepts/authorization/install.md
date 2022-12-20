---
title: PX-Security on Kubernetes clusters
description: Explains how to enable PX-Security in Portworx on an existing kubernetes cluster 
keywords: enable authorization, security, RBAC, Role Based Access Control, JWT, JSON Web Token, self-signed, claims, upgrade to auth, auth enabled cluster
weight: 3000
series: authorization
---

This page guides you to enable the RBAC functionality of PX-Security on an existing Kubernetes cluster. If you are installing a new cluster via the recommended Portworx Operator, see [enable security in Portworx](/cloud-references/security/kubernetes/shared-secret-model-operator/enabling-security/).

## Enable RBAC an existing cluster

If you already have a working Portworx cluster and wish to enhance security by enabling RBAC, you will need to enable it for the entire Portworx cluster.  Depending on how the cluster was installed, follow the steps for either the [Operator-based installation](/cloud-references/security/kubernetes/shared-secret-model-operator/) or the (legacy) [DaemonSet-based installation](/cloud-references/security/kubernetes/shared-secret-model/).

### (Optionally) Generate a new cluster token.

If you use Disaster Recovery functionality or are using data-migrating functionality between Kubernetes clusters, run the following command to generate a new cluster token after these operations, as the token will have changed that is used for for pairing and migrating your clusters:

```text
pxctl cluster token reset
```

You will then need to update any other clusters' [clusterpair objects](/operations/operate-kubernetes/disaster-recovery/px-metro/2-pair-clusters/) with the new token.

## Implications on pxctl

The `pxctl` command will also be secured. As a result, you may need to perform [extra steps](/concepts/authorization/use-pxctl-security-enabled/) to run `pxctl` commands.

## Security parameter overview

The following parameters are utilized and required by PX-Security. With the recommended Operator-based installation, these are automatically created for you, but they can be [manually changed](https://docs.portworx.com/reference/crd/storage-cluster/#security-configuration) if needed.  However, they must be manually [created](/cloud-references/security/kubernetes/shared-secret-model/generating-secrets/) and [provided](/cloud-references/security/kubernetes/shared-secret-model/enabling-security/#using-portworx-daemonset) if you are enabling PX-Security on a legacy DaemonSet-based installation.

### Environment variables (DaemonSet-based installation only)

{{<info>}}
**NOTE:** You do not need to be specify any of these parameters on an Operator-based installation.
{{</info>}}

{{<companyName>}} recommends to providing sensitive information like shared secrets as environment variables. These variables can be provided by _secrets_ through your container orchestration system.

| Environment Variable | Required? | Description |
| -------------------- | --------- | ----------- |
| `PORTWORX_AUTH_SYSTEM_KEY` | Yes | Shared secret used by Portworx to generate tokens for cluster communications |
| `PORTWORX_AUTH_SYSTEM_APPS_KEY` | Yes, when using Stork | Share secret used by Stork to generate tokens to communicate with Portworx. The shared secret must match the value of `PX_SHARED_SECRET` environment variable in Stork. |
| `PORTWORX_AUTH_JWT_SHAREDSECRET` | Optional | Self-generated token shared secret, if any |

### Configuration 

For non-sensitive information, you can use command-line parameters with the following arguments:

| Name | Description |
| ---- | ----------- |
| `-jwt_issuer <issuer>` | JSON Web Token issuer (e.g. openstorage.io). This is the token issuer for your self-signed tokens. It must match the `iss` value in token claims |
| `-jwt_rsa_pubkey_file <file path>` | JSON Web Token RSA Public file path |
| `-jwt_ecds_pubkey_file <file path>` | JSON Web Token ECDS Public file path |
| `-username_claim <claim>` | Name of the claim in the token to be used as the unique ID of the user (<claim> can be `sub`, `email` or `name`, default: `sub`) |
