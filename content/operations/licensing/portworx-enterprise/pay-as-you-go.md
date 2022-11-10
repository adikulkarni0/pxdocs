---
title: Switch to pay-as-you-go billing on an existing Kubernetes cluster
linkTitle: Enable pay-as-you-go billing
keywords: pay-as-you-go, PAYG, 
description: Learn how you can enable pay-as-you-go billing for Portworx.
weight: 300 
aliases:
  - /reference/knowledge-base/licensing/pay-as-you-go/
  - /reference/licensing/pay-as-you-go/
---

If you have already set up your cluster using any of the paid, trial, or free Portworx licenses, you can switch to pay-as-you-go billing by acquiring a pay-as-you-go account key from the Pure Storage support team and performing the following steps.

{{<info>}}
 **NOTE:** For enabling pay-as-you-go billing on your air-gapped cluster, refer to [this page](/operations/licensing/portworx-enterprise/pay-as-you-go-air-gapped).
 {{</info>}}

## Prerequisites

* A pay-as-you-go account key, acquired by contacting the Pure Storage support team.

## Enable pay-as-you-go billing

{{<info>}}
 **NOTE:** The examples below use the `portworx` namespace, you should update this to the correct namespace for your environment.
 {{</info>}}

1. Create a Kubernetes secret and place your pay-as-you-go account key into it:
   
    ```text
    kubectl create secret generic px-saas-key --from-literal=account-key=<PAY-AS-YOU-GO-KEY> -n portworx
    ```

2. Patch the pay-as-you-go secret into your cluster. Follow the steps appropriate for your cluster's install method:

    * **Operator-based installations:**
        
        1. Patch the StorageCluster `stc` with the following command:
            
            ```text
            stc=$(kubectl get stc -n portworx -o jsonpath='{.items[0].metadata.name}')
            ```

        2. Patch the stc using the secret you created in step 1:

            ```text
            kubectl  patch stc $stc  --type='json' -p='[{"op": "add", "path": "/spec/env/0","value": {"name": "SAAS_ACCOUNT_KEY_STRING", "valueFrom":{"secretKeyRef":{"key": "account-key", "name": "px-saas-key"}}}}]' -n portworx
            ```
            The above step patches your StorageCluster `stc` and appends the following under the `env:` parameter. This will enable pay-as-you-go billing on your cluster.
            
             <!--The command is equivalent to manually editing your StorageCluster spec using `kubectl edit stc <px-cluster-ID> -n portworx` -->.

            ```    
            env:
            - name: "SAAS_ACCOUNT_KEY_STRING"
              valueFrom:
                secretKeyRef:
                  name: px-saas-key
                  key: account-key
            ``` 
        
        3. Wait for a few minutes and then verify if the automatic rolling upgrade is successful by running the following command. You will observe that the new pods have been created recently with new `px-cluster-<id>`. You can also see the latest timestamp under `AGE`, as shown in the following example:

            ```text
            kubectl get pods -n portworx -l name=portworx
            ```
            ```output
            NAME                                                    READY   STATUS    RESTARTS   AGE
            px-cluster-60403120-fdec-470b-af4e-ef93c42d8aeb-b66td   2/2     Running   0          6m56s
            px-cluster-60403120-fdec-470b-af4e-ef93c42d8aeb-fqdgj   2/2     Running   0          5m16s
            px-cluster-60403120-fdec-470b-af4e-ef93c42d8aeb-t8ldc   2/2     Running   0          9m
            ```


        4. Check your SaaS key license usage by running `pxctl status` from one of the worker nodes:
        
            ```text
            pxctl status
            ```
            ```output
            Status: PX is operational
            Telemetry: Disabled or Unhealthy
            Metering: Healthy
            License: PX-Enterprise Usage Based (expires in 209 days)
            ...
        
            ```  
            To see more detailed information about your license run `pxctl license list`

            ```text
            pxctl license list
            ```
            ```output
            DESCRIPTION                                             ENABLEMENT                      ADDITIONAL INFO
            Number of nodes maximum                                           1000
            Number of volumes per cluster maximum                            16384
            Volume capacity [TB] maximum                                        40
            Node disk capacity [TB] maximum                                    256
            Node disk capacity extension                                       yes
            Number of snapshots per volume maximum                              64
            Number of attached volumes per node maximum                        256
            Storage aggregation                                                yes
            Shared volumes                                                     yes
            Volume sets                                                        yes
            BYOK data encryption                                               yes
            Limit BYOK encryption to cluster-wide secrets                       no
            Resize volumes on demand                                           yes
            Snapshot to object store [CloudSnap]                               yes
            Number of CloudSnaps daily per volume maximum                    unlim
            Cluster-level migration [Kube-motion/Data Migration]               yes
            Disaster Recovery [PX-DR]                                          yes
            Autopilot Capacity Management                                      yes
            OIDC Security                                                      yes
            Bare-metal hosts                                                   yes
            Virtual machine hosts                                              yes
            Product SKU                                             PX-Enterprise Usage Based       expires in 209 days

            LICENSE EXPIRES: 2023-06-01 23:59:59 +0000 UTC
            For information on purchase, upgrades and support, see
            https://docs.portworx.com/knowledgebase/support.html
            ```

    * **Daemonset-based installations:**
		
        Patch the daemonset using the secret you created in step 1:

        ```text
        kubectl patch ds portworx --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/env/0","value": {"name": "SAAS_ACCOUNT_KEY_STRING", "valueFrom":{"secretKeyRef":{"key": "account-key", "name": "px-saas-key"}}}}]' -n portworx
        ```

###  How to apply a new pay-as-you-go key secret

If you need to update your pay-as-you-go key secret, you can export the secret object to a YAML file, apply the secret, and then perform a rolling update by adding a dummy `env` variable to your StorageCluster `stc`:

2. Export the YAML file:
    
    ```text
    kubectl get secret/px-saas-key -n portworx -o yaml > px-saas-key.yaml
    ```
            
3. After editing the file with your new `pay-as-you-go` key secret, run the `kubectl apply` command:

    ```text
     kubectl apply -f px-saas-key.yaml
    ```
4. Get your `px-cluster ID` using `pxctl status`, and run the following command to trigger the Portworx pod rolling update:

    ```text
    kubectl patch stc <px-cluster-ID> -n portworx --type "json" -p '[{"op":"add","path":"/spec/env/-","value":{"name":"DUMMYENV","value":"date"}}]'
    ```
   
