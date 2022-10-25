---
title: Supported Kernels
keywords: Supported kernels, Install, Configure, Prerequisites
description: Various kernels that are supported by Portworx and steps to install headers
weight: 800
linkTitle: Supported Kernels
series: reference
aliases:
  - /reference/knowledge-base/supported-kernels/
---

Portworx runs as a Docker or OCI container, available on the DockerHub. Portworx has a dependency on the kernel module, which must be installed on hosts. Portworx is distributed with pre-built kernel modules for select Centos and Ubuntu Linux distributions. If your kernel version is not listed in the table below, Portworx attempts to download kernel headers to compile it’s kernel module. This can fail if the host sits behind a proxy.  If you already have the kernel-headers and kernel-devel packages installed the module will compile successfully.  If you do not have the packages you will need to first install them before restarting Portworx.  

To install the kernel headers and kernel development packages for kernels not listed in the table below.

#### CentOS

```text
yum install kernel-headers-`uname -r`
yum install kernel-devel-`uname -r`

```

#### Ubuntu

```text
apt install linux-headers-$(uname -r)

```

## Qualified distros and kernel versions

### 2.12

{{<automateTable source="qualifiedLinuxDistros">}}

{{<automateTable source="cloudLinuxDistros">}}

### 2.11

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
| CentOS 7.8-vanilla | 3.10.0-1127.el7.x86_64 | 
| CentOS 8.2-vanilla | 4.18.0-193.el8.x86_64 | 
| Ubuntu 18.04.5 LTS | 5.4.0-1040-gcp | 
| Ubuntu 20.04.02 LTS | 5.4.0-73-generic |  
| Ubuntu 20.10 | 5.13.0-1028-gcp |  
| Ubuntu 21.04 | 5.11.0-1007-gcp | 
| Fedora 27 | Up to 4.18.19-100.fc27.x86_64 | 
| Fedora 28 | Up to 5.0.16-100.fc28.x86_64 | 
| Fedora 33 | Up to 5.14.18-100.fc33.x86_64 | 
| FlatCar Alpha | 5.10.93-flatcar | 
| FlatCar Beta | 5.10.93-flatcar | 
| FlatCar Stable | 5.10.107-flatcar |  
| RHEL 7.6 | 4.20.13-1.el7.elrepo.x86_64 |  
| RHEL 7.9 | Up to 3.10.0-1127.el7.x86_64 | 
| RHEL 8.4 | Up to 4.18.0-305.25.1.el8_4.x86_64 | 
| RHEL 8.5 (Ootpa) | Up to 4.18.0-372.16.1.el8_6.x86_64 | 
| Debian 9 | Up to 4.9.0-13-amd64 |  
| Debian 10 | Up to 4.19.0-20-cloud-amd64 | 
| OEL 7.9 | Up to 3.10.0-1160.59.1.el7.x86_64 | 

| **Cloud Distro** | **Kernel Version** |
| --- | --- |
| Amazon Linux v2 | 4.14.232-177.418.amzn2.x86_64 | 
| Photon 3.0 | 4.19.132-6.ph3 | 
| Photon 4.0 | 5.10.4-16.ph4 | 

### 2.10

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
| CentOS 7.5 | 5.4.12-1.el7.elrepo.x86_64 |
| CentOS 7.8-vanilla | 3.10.0-1127.el7.x86_64 |
| CentOS 8.2-vanilla | 4.18.0-240.22.1.el8_3.x86_64 |
| Ubuntu 18.04.5 LTS | 5.4.0-1040-gcp |
| Ubuntu 20.04.02 LTS | 5.4.0-73-generic | 
| Ubuntu 20.10 | 5.13.0-1019-gcp | 
| Ubuntu 21.04 | 5.11.0-1007-gcp |
| Fedora 27	| Up to 4.13.9-300.fc27.x86_64 |
| Fedora 28	| Up to 4.16.3-301.fc28.x86_64 |
| FlatCar Alpha | 5.10.93-flatcar |
| FlatCar Beta | 5.10.93-flatcar |
| FlatCar Stable | 5.10.107-flatcar | 
| RHEL 7.6 | 4.20.13-1.el7.elrepo.x86_64 | 
| RHEL 7.8	| Up to 3.10.0-1160.25.1.el7.x86_64 |
| RHEL 8.4 BETA | 4.18.0-293.el8.x86_64 |
| Debian 9	| Up to 4.9.0-15-amd64 | 
| Debian 10 | Up to 4.19.0-16-cloud-amd64 |
| OEL 7.9 | Up to 3.10.0-1160.59.1.el7.x86_64 |


| **Cloud Distro** | **Kernel Version** |
| --- | --- |
| Amazon Linux v2	| 4.14.225-169.362.amzn2.x86_64 |
| Photon 3.0 | 4.19.132-6.ph3 | 
| Photon 4.0 | 5.10.4-16.ph4 | 

### 2.9

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
| CentOS 7.5 | 5.4.12-1.el7.elrepo.x86_64 |
| CentOS 7.8-vanilla | 3.10.0-1127.el7.x86_64 |
| CentOS 8.2-vanilla | 4.18.0-240.22.1.el8_3.x86_64 |
| Ubuntu 18.04.5 LTS | 5.4.0-1040-gcp |
| Ubuntu 20.04.02 LTS | 5.4.0-73-generic | 
| Ubuntu 20.10 | 5.8.0-1031-gcp | 
| Ubuntu 21.04 | 5.11.0-1007-gcp |
| Fedora 27	| Up to 4.13.9-300.fc27.x86_64 |
| Fedora 28	| Up to 4.16.3-301.fc28.x86_64 |
| FlatCar Alpha | 5.10.46-flatcar |
| FlatCar Beta | 5.10.46-flatcar |
| FlatCar Stable | 5.10.75-flatcar | 
| RHEL 7.6 | 4.20.13-1.el7.elrepo.x86_64 | 
| RHEL 7.8	| Up to 3.10.0-1160.25.1.el7.x86_64 |
| RHEL 8.4 BETA | 4.18.0-293.el8.x86_64 |
| Debian 9	| Up to 4.9.0-15-amd64 | 
| Debian 10 | Up to 4.19.0-16-cloud-amd64 |

| **Cloud Distro** | **Kernel Version** |
| --- | --- |
| Amazon Linux v2	| 4.14.225-169.362.amzn2.x86_64 |
| Photon 3.0 | 4.19.132-6.ph3 | 
| Photon 4.0 | 5.10.4-16.ph4 | 

### 2.8

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
| CentOS 7.5 | 5.4.12-1.el7.elrepo.x86_64 |
| CentOS 7.8-vanilla | 3.10.0-1127.el7.x86_64 |
| CentOS 8.2-vanilla | 4.18.0-240.22.1.el8_3.x86_64 |
| Ubuntu 16.04.7 LTS | 4.4.0-116-generic | 
| Ubuntu 18.04.5 LTS | 5.4.0-1040-gcp |
| Ubuntu 20.04.02 LTS | 5.4.0-73-generic | 
| Ubuntu 20.10 | 5.8.0-1031-gcp | 
| Fedora 27	| Up to 4.13.9-300.fc27.x86_64 |
| Fedora 28	| Up to 4.16.3-301.fc28.x86_64 |
| FlatCar Alpha | 5.10.32-flatcar |
| FlatCar Beta | 5.10.32-flatcar |
| FlatCar Stable | 5.10.32-flatcar | 
| RHEL 7.6 | 3.10.0-1160.25.1.el7.x86_64 | 
| RHEL 7.8	| Up to 3.10.0-1160.25.1.el7.x86_64 |
| RHEL 8.4 BETA | 4.18.0-293.el8.x86_64 |
| Debian 9	| Up to 4.9.0-15-amd64 | 
| Debian 10 | Up to 4.19.0-16-cloud-amd64 |

| **Cloud Distro** | **Kernel Version** |
| --- | --- |
| Amazon Linux v2	| 4.14.225-169.362.amzn2.x86_64 |
| Photon 3.0 | 4.19.132-6.ph3 | 
| Photon 4.0 | 5.10.4-16.ph4 | 

### 2.7

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
| CentOS 7.5 | 5.4.12-1.el7.elrepo.x86_64 |
| CentOS 7.6 | 5.7.7-1.el7.elrepo.x86_64, 5.7.12-1.el7.elrepo.x86_64 | <!-- Missing  --> 
| CentOS 7.8-vanilla | 3.10.0-1127.el7.x86_64 |
| CentOS 8.2-vanilla | 4.18.0-193.el8.x86_64 |
| Ubuntu 16.04 | 4.15.0-142-generic, 4.15.0-144-generic |
| Ubuntu 18.04.5 LTS | 4.15.0-135-generic, 5.4.0-1040-gcp, 5.4.0-1036-azure, 5.3.0-1039-gke, IBM Cloud: 4.15.0-142 |
| Ubuntu 20.04 LTS| 5.4.0-72-generic |
| Ubuntu 20.10 | 5.8.0-1028-gcp | <!-- New -->
| Fedora 27	| Up to 4.13.9-300.fc27.x86_64 |
| Fedora 28	| Up to 5.0.16-100.fc28.x86_64|
| FlatCar Alpha | 5.10.25-flatcar |
| FlatCar Beta | 5.10.25-flatcar |
| FlatCar Stable | 5.10.25-flatcar | <!-- New -->
| RHEL 7.8	| Up to 3.10.0-1160.24.1.el7.x86_64, IBM Cloud: 3.10.0-1160.24.1.el7 |
| RHEL 8.2	| 4.18.0-193.29.1.el8_2.x86_64, 4.18.0-193.el8_2.x86_64 |
| RHEL 8.3 (Ootpa) | 4.18.0-240.1.1.el8_3.x86_64, 4.18.0-240.15.1.el8_3.x86_64 | <!-- New -->
| RHEL 8.4 BETA | 4.18.0-293.el8.x86_64 | <!-- New -->
| Debian 9	| Up to 4.9.0-14-amd64 | <!-- Missing  --> 
| Debian 10 | Up to 4.19.0-16-cloud-amd64 |

| **Cloud Distro** | **Kernel Version** |
| --- | --- |
| Amazon Linux v2	| 4.14.219-164.354.amzn2.x86_64 |




### 2.6

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
| CentOS 7.5 | 5.4.12-1.el7.elrepo.x86_64 |
| CentOS 7.8-vanilla | 3.10.0-1127.el7.x86_64 |
| CentOS 8.2-vanilla | 4.18.0-240.10.1.el8_3.x86_64 |
| Ubuntu 16.04	| Up to 4.4.0-116-generic |
| Ubuntu 2004 | 5.4.0-42-generic |
| Fedora 27	| Up to 4.13.9-300.fc27.x86_64 |
| Fedora 28	| Up to 5.0.16-100.fc28.x86_64 |
| RHEL 7.5	| Up to 3.10.0-1127.el7.x86_64 |
| RHEL 7.6	| Up to 3.10.0-1127.el7.x86_64 |
| RHEL 7.8	| Up to 3.10.0-1127.el7.x86_64 |
| RHEL 8.2	| 4.18.0-240.10.1.el8_3.x86_64 |
| Debian 9	| Up to 4.9.0-12-amd64 |
| FlatCar Stable | 5.4.77-flatcar |
| FlatCar Beta | 5.8.18-flatcar |

| **Cloud Distro** | **Kernel Version** |
| --- | --- |
| CoreOS Alpha	| 4.9.123-coreos |
| CoreOS Beta 2411.1.0 (Rhyolite)	| 4.9.123-coreos |
| CoreOS Stable 2345.3.0 (Rhyolite)	| 4.9.123-coreos |
| Amazon Linux v2	| 4.14.186-146.268.amzn2.x86_64 |

<!-- 
| RHEL 8.1 (Ootpa)	|  |
| Ubuntu19.04	|  |
| Ubuntu18.04.4 LTS	|  |
-->


### 2.5.3

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
| Ubuntu 16.04	| Up to 4.4.0-116-generic #140-Ubuntu |
| Fedora 27	| Up to 4.18.18-100.fc27.x86_64 |
| Fedora 28	| Up to 4.18.10-200.fc28.x86_64 |
| RHEL 7.5	| Up to 3.10.0-1127.el7.x86_64 |
| RHEL 7.8	| Up to 3.10.0-1127.el7.x86_64 |
| Debian 9	| Up to 4.9.0-12-amd64 #1 SMP Debian 4.9.210-1 |

| **Cloud Distro** | **Kernel Version** |
| --- | --- |
| CoreOS Alpha	| Up to 4.19.106-coreos |
| CoreOS Beta 2411.1.0 (Rhyolite)	| Up to 4.19.106-coreos |
| CoreOS Stable 2345.3.0 (Rhyolite)	| Up to 4.19.106-coreos |
| RHEL 8.1 (Ootpa)	| Up to 4.18.0-80.4.2.el8_0.x86_64 |
| Ubuntu19.04	| Up to 5.0.0-1017-gcp |
| Ubuntu18.04.4 LTS	| Up to 5.0.0-1033-gcp |
| Amazon Linux v2	| Up to 4.14.77-81.59.amzn2.x86_64 |


### 2.0.0

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
|CentOS/RHEL | Up to 3.10.0-957.el7.x86_64 |
|Ubuntu | Up to 4.19.5-041905-generic |

### 1.7.2

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
| CentOS/RHEL| Up to 3.10.0-957.el7.x86_64 |

### 1.7.1.1

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
| CentOS/RHEL| Up to 3.10.0-862.14.4.el7.x86_64 |
|Ubuntu | Up to 4.19.1-041901-generic |

### 1.7.1

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
|CentOS/RHEL| Up to 3.10.0-862.14.4.el7.x86_64 |
|Ubuntu| Up to 4.19.0-041900-generic |

### 1.7.0

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
|CentOS/RHEL| Up to 3.10.0-862.14.4.el7.x86_64 |
|Ubuntu| Up to 4.19.0-041900-generic |

### 1.6.1.4 and 1.6.1.3

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
|CentOS/RHEL| Up to 3.10.0-862.14.4.el7.x86_64 |
|Ubuntu| Up to 4.19.0-041900-generic |

### 1.6.1.2

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
|CentOS/RHEL| Up to 3.10.0-862.14.4.el7.x86_64 |
|Ubuntu| Up to 4.18.16-041816-generic |

### 1.6.1.1

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
|CentOS/RHEL| Up to 3.10.0-862.14.4.el7.x86_64 |
|Ubuntu| Up to 4.18.12-041812-generic |

### 1.6.1

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
|CentOS/RHEL| Up to 3.10.0-862.14.4.el7.x86_64 |
|Ubuntu|Up to 4.18.11-041811-generic |

### 1.6.0

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
|CentOS/RHEL| Up to 3.10.0-862.11.6.el7.x86_64 |
|Ubuntu|Up to 4.18.8-041808-generic |

### 1.5.1

| **Linux Distro**	| **Kernel Version** |
| --- | --- |
|CentOS/RHEL| Up to 3.10.0-862.11.6.el7.x86_64 |
|Ubuntu|Up to 4.18.7-041807-generic |
