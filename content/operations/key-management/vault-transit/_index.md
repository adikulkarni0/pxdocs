---
title: Vault Transit
logo: /logos/hashicorp-vault.png
keywords: Vault Key Management, Vault Transit, Hashicorp, encryption keys, secrets, Volume Encryption, Cloud Credentials, secret store, passwords
description: Instructions on using Vault key management with Portworx
weight: 500
disableprevnext: true
series: key-management
noicon: true
---

Portworx can be integrated with Vault Transit to encrypt volumes. This page guides you to connect a Portworx cluster to a [Vault development](/operations/key-management/vault/#set-up-vault) server and enable Vault Transit, which can be used to store secrets for encrypting volumes.

## What is Vault Transit?
Vault Transit manages key generation for in-transit data encryption. With Vault Transit, you do not need to set a cluster wide secret to encrypt volumes and PVCs. By default, Portworx uses generated keys from Vault Transit as passphrase for volume encryption.

## Prerequisites

* [Vault is installed](/operations/key-management/vault/#set-up-vault)
* [Vault environment is configured](/operations/key-management/vault/#set-up-the-vault-development-environment)


## Configure Vault Transit environment

1. Run the following command to enable the Transit secrets engine:

  ```text
  vault secrets enable transit
  ```

2. If you configured Vault strictly with policies, then the Vault Transit token provided to Portworx should follow the following policies:

  ```
  # Enable transit secrets engine
  path "sys/mounts/transit" {
    capabilities = [ "create", "read", "update", "delete", "list" ]
  }

  # To read enabled secrets engines
  path "sys/mounts" {
    capabilities = [ "read" ]
  }

  # Manage the transit secrets engine
  path "transit/*" {
    capabilities = [ "create", "read", "update", "delete", "list" ]
  }

  # Read and List capabilities on mount to determine which version of kv backend is supported
  path "sys/mounts/*"
  {
  capabilities = ["read", "list"]
  }

  # V1 backends (Using default backend)
  # Provide full access to the portworx subkey
  path "secret/*"
  {
  capabilities = ["create", "read", "update", "delete", "list"]
  }

  # V1 backends (Using custom backend)
  # Provide full access to the portworx subkey
  # Provide -> VAULT_BACKEND_PATH=custom-backend (required)
  path "custom-backend/*"
  {
  capabilities = ["create", "read", "update", "delete", "list"]
  }


  # V2 backends (Using default backend )
  # Provide full access to the data/portworx subkey
  path "secret/data/*"
  {
  capabilities = ["create", "read", "update", "delete", "list"]
  }

  # V2 backends (Using custom backend )
  # Provide full access to the data/portworx subkey
  # Provide -> VAULT_BACKEND_PATH=custom-backend (required)
  path "custom-backend/data/*"
  {
  capabilities = ["create", "read", "update", "delete", "list"]
  }
  ```


## Set the Vault Transit secrets engine for Portworx

Depending on whether you are performing a fresh install or modifying an existing installation, proceed to one of the following sections.

### New Installation

When generating the [Portworx Kubernetes specification file](https://central.portworx.com), select **Vault Transit** from the **Secrets Store Type** dropdown menu of **Advanced Settings** on the **Customize** tab.


### Existing Installation

Depending on the type of your Portworx installation, proceed to one of the following sections.

#### Operator

Edit the `StorageCluster` object by setting the value of the `specs.secretsProvider` field to `vault-transit`.

```text
spec:
  secretsProvider: vault-transit
```

#### DaemonSet

Set the `secret_type` field to `vault-transit`, so that all the Portworx nodes start using Vault Transit:

```text
kubectl edit daemonset portworx -n kube-system
```

Add `-secret_type` and `vault-transit` arguments to the portworx container in the DaemonSet, as shown in the following example:

```text
containers:
  - args:
    - -c
    - testclusterid
    - -s
    - /dev/sdb
    - -x
    - kubernetes
    - -secret_type
    - vault-transit
    name: portworx
```

Editing the DaemonSet or Operator spec will restart all Portworx pods.


### Authenticate Portworx

Use one of the [supported methods](/operations/key-management/vault/#kubernetes-users) to authenticate Portworx with Vault Transit.


#### (Optional) Customize the key path

Vault Transit generates the keys by writing to a `transit` key path. For example:

```
$ vault write -f transit/keys/my-key
Success! Data written to: transit/keys/my-key
```

By default, Portworx uses transit key path `pwx-encryption-key` (full path: `transit/keys/pwx-encryption-key`) for key generation. To customize the key path with Vault Transit, specify the path as a base64 encoded string in `px-vault` Secret object.

```
VAULT_ENCRYPTION_KEY: pwx-test-key
```

{{<info>}}
**NOTE:** Portworx does not recommend changing the value of `VAULT_ENCRYPTION_KEY` once deployed as the previous secret keys and volumes might be inoperative if the key path is changed.
{{</info>}}


## Set cluster wide secret key (Optional)

A cluster wide secret key is a common key that you can use to encrypt all your volumes. Run the following command to set the cluster secret key:

```text
pxctl secrets set-cluster-key --secret <cluster-wide-secret-key>
```

You should run this command only once for the cluster. If you added the cluster secret key through the `config.json`, then the above command overwrites it. Even on subsequent Portworx restarts, the cluster secret key in `config.json` will be ignored.

## Use Vault Transit with Portworx

{{<homelist series="vault-transit-secret-uses">}}
