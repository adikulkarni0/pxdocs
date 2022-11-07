---
title: Portworx CSI Driver
description: Find out more about the Portworx CSI Driver and discover documentation for the different container schedulers we support
keywords: portworx, kurbernetes, containers, storage, csi
weight: 400
hidesections: true
noicon: true
disableprevnext: true
scrollspy-container: false
series: operations
aliases:
    - /portworx-csi-driver/
---
The Portworx CSI Driver has been verified on both [Kubernetes](/operations/operate-kubernetes/storage-operations/csi/) and [Nomad](/install-portworx/install-with-other/nomad/). The CSI Driver supports all existing Portworx features as well as most CSI features. See the table below for a detailed picture of what features we support for each scheduler.

For scheduler-specific information, refer to the following pages:

* [CSI Driver on Kubernetes](/operations/operate-kubernetes/storage-operations/csi/)
* [CSI Driver on Nomad](/install-portworx/install-with-other/nomad//)

## CSI-based Installation Options

| **Install Workflow**                | **Kubernetes**                                                              | **Nomad** |
|-------------------------------------|--------------------------------------------------|--------------------------------------|
| Operator                     | Yes                                                     | No  |
| Portworx Daemonset Installer | Yes                                                     | No  |
| Nomad Job                    | No                                                      | [Yes](/install-portworx/install-with-other/nomad/installation/install-as-a-nomad-job/) | 

## Core Features support
| **Feature**  | **Kubernetes** | **Nomad** |
|----------|------------|-------|
| Volume Lifecycle               | Yes                                    | Yes |
| Sharedv4 Volumes               | Yes                                    | Yes |
| Volume Cloning                 | Yes                                    | Yes |
| CSI Snapshotting               | Yes                                    | Yes |
| CSI Spec 1.6                   | Yes                                    | Yes |
| PX Security - Volume Lifecycle | Yes                                    | Yes |
| PX Security - Snapshotting     | Yes                                    | Yes |
| PX Topology                    | Yes                                    | No  |
| CSI Raw Block                  | Yes                                    | No  |
| Volume Resizing                | Yes                                    | Not supported by Nomad: see [#10324](https://github.com/hashicorp/nomad/issues/10324) |
| Encryption - Vault             | Yes                                    | Not supported by Nomad: see [#7978](https://github.com/hashicorp/nomad/issues/7978)   |
| CSI Topology                   | No; [Portworx topology](/concepts/update-geography-info/) is recommended instead | No  |

## Kubernetes specific features

| **Feature**                         | **Supported**                                                              |
|-------------------------------------|----------------------------------------------------------------------------|
| Volume Placement Strategies             | Yes                                                                    |
| Auth Token Secrets                      | Yes                                                                    |
| Autopilot                               | Yes                                                                    |
| Generic Ephemeral Volumes               | Yes                                                                    |
| Encryption with Kubernetes secrets      | Yes                                                                    |
| CSI Migration (Portworx in-tree -> CSI) | In development                                                         |

## Nomad specific features

| **Feature**                         | **Supported**                                                              |
|-------------------------------------|----------------------------------------------------------------------------|
| Pre-provisioned volume registration | Yes                                                                        |

## Portworx Backup & Stork

| **Feature**                         | **Kubernetes**                                                              | **Nomad** |
|-------------------------------------|--------------------------------------------------|--------------------------------------|
| Portworx Backup support | Yes                                                     | No |
| DR/Disaster Recovery    | Yes                                                     | No |
| Stork Snapshotting      | Yes                                                     | No |
| Stork Migration         | Yes                                                     | No |
| Stork Backup            | Yes                                                     | No |
