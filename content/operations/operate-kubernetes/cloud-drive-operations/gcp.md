---
title: GCP cloud drive encryption with customer managed encryption keys
linkTitle: GCP cloud drive encryption with CMEK
weight: 100
keywords: gcp, gce, gke, cloud drives, encryption, customer managed key, cmek
description: Learn how to encrypt GCP cloud drives with customer managed keys.
---

This page describes how to encrypt Google Cloud Platform (GCP) cloud drives with customer managed keys using Google Key Management Service (KMS).

## Create a disk encryption key

### Create a KMS key ring 

1. Navigate to [Key Management](https://console.cloud.google.com/security/kms/keyrings) in the Google Cloud console.

2. Click **Create Key Ring**.

3. Provide a Key Ring name, choose your region, and click **Create**.

    {{<info>}}
**NOTE**: Choose either the same region as your cluster or the Multi-region **global** region. Keys from different regions should not be used for disk encryption.
    {{</info>}}

### Create a symmetric key

1. Click the Key Ring that you created in the previous section.

2. Click **Create key**.

3. Provide a key name, choose **Symmetric encrypt/decrypt** for **Purpose**, select an automatic rotation policy, and click **Create**.

4. To get the name of the resource that you need to provide for disk encryption, click the **Actions** menu<span class="material-icons">more_vert</span> for your key, then click **Copy resource name**.<br><br>

    The key is in the following format:

    ```text
    projects/{{<highlight>}}<projectName>{{</highlight>}}/locations/{{<highlight>}}<region>{{</highlight>}}/keyRings/{{<highlight>}}<keyRing>{{</highlight>}}/cryptoKeys/{{<highlight>}}<keyName>{{</highlight>}}
    ```

{{<info>}}
**WARNING**: Google KMS lets you set an automatic key rotation policy for your KMS key, and it creates a new key version at each scheduled rotation. Do not disable or delete old key versions after key rotation is complete.

If you mark a key for deletion, the key version stays scheduled for destruction for a default period of 24 hours or a configured duration, after which it is automatically destroyed. _Any data encrypted with this key version is not recoverable._
{{</info>}}

## Enable a KMS account for disk encryption

A service account is required for performing encryption operations with Google KMS.

This service account must have the [Cloud KMS CryptoKey Encrypter/Decrypter 
(`roles/cloudkms.cryptoKeyEncrypterDecrypter`)](https://cloud.google.com/kms/docs/reference/permissions-and-roles#cloudkms.cryptoKeyEncrypterDecrypter) role. You can either make a separate service account that is responsible for disk encryption and decryption operations, or you can add the role to the default service account managing Portworx.


### Use a separate KMS service account

#### Create and configure a KMS service account

1. Navigate to the [Service Accounts page of the Google Cloud console](https://console.cloud.google.com/iam-admin/serviceaccounts).

2. Click **Create service account**.

3. Under **Service account details**, provide an ID for your service account, then click **Create and continue**.

3. Under **Grant this service account access to project**, in the **Select a role** menu, select the **Cloud KMS CryptoKey Encrypter/Decrypter** role, then click **Continue**.

4. Under **Grant users access to this service account**, in the **Service account users role** field, enter the default service account that you use to manage Portworx, then click **Done**.

{{<info>}}
**WARNING**: A KMS service account that has been used to encrypt a disk must be enabled for the life cycle of that disk. If a KMS service account is deactivated or deleted, any data that was encrypted with that particular KMS service account cannot be retrieved.
{{</info>}}

#### Modify your StorageCluster spec

In your StorageCluster spec, specify your KMS key in front of every cloud drive as follows:

```text
cloudStorage:
    deviceSpecs:
    - type=pd-standard,size=150,kms={{<highlight>}}<kms-key>{{</highlight>}},kmsAccount={{<highlight>}}<kmsServiceAccount>{{</highlight>}}
```

Where:

* `<kms-key>` is the key resource name in the following format:

    ```text
    projects/{{<highlight>}}<projectName>{{</highlight>}}/locations/{{<highlight>}}<region>{{</highlight>}}/keyRings/{{<highlight>}}<keyRing>{{</highlight>}}/cryptoKeys/{{<highlight>}}<keyName>{{</highlight>}}
    ```

* `<kmsServiceAccount>` is the name of the new service account that you created, in the following format:

    ```text
    {{<highlight>}}<serviceAccountName>{{</highlight>}}@{{<highlight>}}<project>{{</highlight>}}.iam.gserviceaccount.com
    ```

### Use the default Portworx service account

1. Navigate to the [IAM page of the Google Cloud console](https://console.cloud.google.com/iam-admin/iam).

2. Under **View by principals**, select the default service account that you use to manage Portworx.

3. Click **Edit principals** <span class="material-icons">edit</span>.

4. Under **Assign roles**, click **Add a role** or **Add another role** and choose the **Cloud KMS CryptoKey Encrypter/Decrypter** role, then click **Save**.

5. In your StorageCluster spec, specify your KMS key in front of every cloud drive as follows:

    ```text
    cloudStorage:
        deviceSpecs:
        - type=pd-standard,size=150,kms={{<highlight>}}<kms-key>{{</highlight>}}
    ```

    Where `<kms-key>` is the resource name in the following format:

    ```text
    projects/{{<highlight>}}<projectName>{{</highlight>}}/locations/{{<highlight>}}<region>{{</highlight>}}/keyRings/{{<highlight>}}<keyRing>{{</highlight>}}/cryptoKeys/{{<highlight>}}<keyName>{{</highlight>}}
    ```

## Verify your drive encryption

There are two ways to check the encryption for your cloud drives.

### From pxctl

Execute the following command:

```text
pxctl clouddrive list
```
```output
px-cloud-drive-a08e9055-3c26-4533-893a-29330f3de598(data)(cmk)
```

Disks which are encrypted with customer managed keys include `(cmk)`, as in the above output.

### From the Google Cloud console

1. Navigate to the [Compute Engine](https://console.cloud.google.com/compute/instances) page of the Google Cloud console.

2. Click the name of any of the nodes in your cluster that should have attached disks that are encrypted with CMEK.<br><br>

    Under **Additional disks**, in the **Encryption** column, disks which are encrypted with customer managed keys show as **Customer-managed**.