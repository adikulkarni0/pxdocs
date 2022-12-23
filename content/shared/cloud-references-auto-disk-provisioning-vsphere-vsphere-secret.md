---
title: Kubernetes secret for VMware
description: Kubernetes secret for VMware
keywords: Kubernetes secrets, Automatic Disk Provisioning, Dynamic Disk Provisioning, VMWare, vSphere ASG, Kubernetes, k8s
hidden: true
---


Update the following items in the Secret template below to match your environment:

* **VSPHERE_USER**: Use output of `echo '<vcenter-server-user>' | base64`
* **VSPHERE_PASSWORD**: Use output of `echo '<vcenter-server-password>' | base64`

   ```text
   apiVersion: v1
   kind: Secret
   metadata:
     name: px-vsphere-secret
     namespace: kube-system
   type: Opaque
   data:
     VSPHERE_USER: XXXX
     VSPHERE_PASSWORD: XXXX
   ```

`kubectl apply` the above spec after you update the above template with your user and password.
