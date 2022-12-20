---
title: Secure your storage with the Operator
description: Simple security setup using shared secrets and leveraging user authentication observed by Kubernetes
keywords: portworx, kubernetes, security, jwt, secret, sharedsecret
weight: 1000
series: ra-kubernetes-security
---

# Overview

While Kubernetes provides a great authentication model for its users, storage
systems could be exposed to malicious requests. PX-Security provides a
method to protect against such requests, further providing deployers with a more secured system.

The following reference architecture describes how to setup PX-Security
to authenticate PVC requests from Kubernetes. This model leverages Kubernetes
user authentication, which secures access to Namespaces, Secrets, and
PersistentVolumes. With access already provided and secured by Kubernetes,
this reference architecture provides a model to secure the communication
between Kubernetes and Portworx. Securing Portworx also protects the storage
system from unwanted access from outside Kubernetes.

Perform the steps in the following sections to set up PX-Security according to this reference architecture:

# Prerequisites

* Your Portworx cluster must be using an Operator-based installation (not DaemonSet)
* You must be running Portworx version 2.1 or greater on Kubernetes
* You must have Operator version 1.4 or greater
