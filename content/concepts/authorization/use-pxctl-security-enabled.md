---
title: "Use pxctl with security enabled"
description: "Explain how pxctl needs security context setup with rbac enabled"
keywords: generate, token, jwt, pxctl, authorization, security, pxctl, context
weight: 8000
series: authorization
---

Once a storage cluster with PX-Security enabled is running, a cluster admin must set up a [`pxctl` context](/reference/cli/authorization/) on each node in order to interact with the system.

The following steps will guide an Operator-based storage admin to setup `pxctl` contexts on each node.

1. Retrieve the admin token from the namespace in which Portworx was installed and store it in th
    ```text
    ADMIN_TOKEN=$(kubectl -n kube-system get secret px-admin-token --template='{{index .data "auth-token" | base64decode}}')
    ```

2. Find the Portworx pod that is running on the node in which the admin wants to interact with:

    ```
    K8_NODE_NAME=kubernetes-worker-3.mylab.lan # must match what appears in output of kubectl get nodes
    PX_POD=$(kubectl -n kube-system get pods -l name=portworx -o jsonpath='{.items[?(@.spec.nodeName == "'K8_NODE_NAME'")].metadata.name}')
    ```

3. Save the admin token in the `pxctl` [context](/reference/cli/authorization/#contexts) for that pod:

    ```text
    kubectl -n kube-system exec -ti $PX_POD -- /opt/pwx/bin/pxctl context create admin --token=$ADMIN_TOKEN
    ```

4. Use `kubectl exec` to access the Portworx container and perform any `pxctl` operations:

    ```
    kubectl -n kube-system exec -ti $PX_POD -- /opt/pwx/bin/pxctl status
    ```

{{<info>}}
**Note:**
This `pxctl` context will need to be refreshed every time the token expires. This is 24 hours by default, but this default can be changed. See [customizing security](/cloud-references/security/kubernetes/shared-secret-model-operator/customizing-security/#changing-token-lifetime) for more information.
{{</info>}}
