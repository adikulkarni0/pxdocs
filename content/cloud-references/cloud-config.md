---
title: Cloud Config Reference
description: Cloud configuration reference
keywords: Cloud configuration, Reference, how to
hidden: true
hidesections: true
disableprevnext: true
---

This is the schema definition for a valid yaml file. For AWS, this file is passed in through user-data.

```yaml
portworx:
  config:
    clusterid: "unique_cluster_id"
    mgtiface: "enp5s0f0"
    dataiface: "enp5s0f1"
    kvdb:
    - etcd:http://etcd0.yourdomain.com:4001
    - etcd:http://etcd1.yourdomain.com:4001
    alertingurl: ""
    storage:
      devices:
      - vol-0743df7bf5657dad8
      - vol-0055e5913b79fb49d
```

## Definitions

**clusterid**:   Globally unique cluster ID.  Ex: `"07ea5dc0-4e9a-11e6-b2fd-0242ac110003"`.   Must be either assigned by {{< pxEnterprise >}} or guaranteed to be unique

**mgtiface**:   Host ethernet interface used for Management traffic.  Primarily used for statistics, configuration and control-path.   Ex: `"enp5s0f0"`

**dataiface**:  Host ethernet interface used for backend activity, such as replication and resync.  Ex: `"enp5s0f1"`

**kvdb**:  Array of endpoints used for the key-value database.  Must be reachable and refer to `etcd` or `consul`.

```yaml
 Ex:
    kvdb:
    - etcd:http://etcd0.yourdomain.com:4001
    - etcd:http://etcd1.yourdomain.com:4001
```

**storage**:   Array of devices to be used as part of the Portworx Storage Fabric. This can be either volume-ids for cloud providers or path to attached devices.

```yaml
Ex:
    storage:
      devices:
      - vol-0743df7bf5657dad8
      - vol-0055e5913b79fb49d
```

Or for attached devices:

```yaml
Ex:
    storage:
      devices:
      - /dev/xvdg
      - /dev/xvdh
```
