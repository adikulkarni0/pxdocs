---
title: SchedulePolicy
description: SchedulePolicy CRD reference
keywords: Portworx, operator, SechedulePolicy, CRD, reference
weight: 200
---

You can use a schedule policy to specify when Portworx should trigger a specific operation. Schedule policies do not contain any actions themselves. <!-- Also, they are not namespaced. //TODO: I'm not sure this is true. The migration example seems to be namespaced. -->
Storage policies are similar to storage classes where an admin creates a schedule policy, and then the users consume it.

## SchedulePolicy schema

This section explains the fields you can use to configure a `SchedulePolicy` object.

A `SchedulePolicy` object has the following sections:

* **Interval:** For interval operations, how frequently Portworx will trigger the operation
* **Daily:** For daily operations, Portworx will trigger the operation at the specified time every day
* **Weekly:** For weekly operations, Portworx will trigger the operation at the specified day and time every week
* **Monthly:** For monthly operations, Portworx will trigger the operation at the specified day and time every month

| Field | Description | Type | Default |
|----|----|----|----|
| apiVersion | Specifies which version of the Stork API you're using to create this object | string | stork.<br/>libopenstorage.org/</br>v1alpha1 |
| kind | Specifies the type of the CRD as `SchedulePolicy`| string | |
| metadata.</br>name | Specifies a name for the `SchedulePolicy` object | string | |
| metadata.</br>namespace | Specifies the namespace in which Porworx creates the `SchedulePolicy` object <!-- See my previous comment -->| string | |
| policy.</br>interval.</br>intervalInMintes | Specifies the interval, in minutes, after which Portworx triggers the operation | number | |
| policy.</br>interval.</br>retain | For backup operations, specifies how many backups Portworx will retain | number | 10 |
| policy.</br>daily.</br>time | Specifies the time of the day when Portworx will trigger the operation, in the 12 hour AM/PM format | string |  |
| policy.</br>daily.</br>retain | For backup operations, specifies how many backups Portworx will retain | number | 30 |
| policy.</br>daily.</br>forceFullSnapshotDay | For backup operations, specifies on which day to trigger a full backup. By default, Portworx will perform 6 incremental and 1 full backup. This field allows you to enforce a full backup in the daily policy on a specific day. | string | Monday |
| policy.</br>weekly.</br>day | Specifies the day of the week when Portworx will trigger the operation. You can use both the abbreviated or the full name of the day of the week | string | |
| policy.</br>weekly.</br>time | Specifies the time of the day when Portworx will trigger the operation, in the 12 hour AM/PM format | string | |
| policy.</br>weekly.</br>retain | For backup operations, specifies how many backups Portworx will retain | string | 7 |
| policy.</br>monthly.</br>day | Specifies the day of the month when Portworx will trigger the operation [^1] | number | |
| policy.</br>monthly.</br>time | Specifies the time of the day when Portworx will trigger the operation, in the 12 hour AM/PM format | string | |
| policy.</br>monthly.</br>retain | For backup operations, specifies how many backups Portworx will retain | number | 12 |

[^1]: The date of the month should be greater than 0 and less than 31. If the specified date is invalid, it will roll over to the next month. For example, if you specify the date as Feb 31, Portworx will trigger the operation either on the 2nd or 3rd March, depending on whether a year is a leap year or not.


## SchedulePolicy examples

### Interval SchedulePolicy

An interval `SchedulePolicy` includes the following fields and values:

* **apiVersion:** the version of the Stork scheduler (this example uses `stork.libopenstorage.org/v1alpha1`)
* **kind:** as `SchedulePolicy`
* **metadata.name:** the name of the `SchedulePolicy` object (this example uses `testpolicy`)
* **metadata.namespace:** the name of your namespace (this example uses `mysql`)
* **policy.interval.intervalInMinutes:** the interval, in minutes, after which Portworx triggers the operation (this example triggers the operation every minute)

    ```text
    apiVersion: stork.libopenstorage.org/v1alpha1
    kind: SchedulePolicy
    metadata:
      name: testpolicy
      namespace: mysql
    policy:
      interval:
        intervalMinutes: 1
    ```

### Daily SchedulePolicy

A daily `SchedulePolicy` includes the following fields and values:

* **apiVersion:** the version of the Stork scheduler (this example uses `stork.libopenstorage.org/v1alpha1`)
* **kind:** as `SchedulePolicy`
* **metadata.name:** the name of the `SchedulePolicy` object (this example uses `testpolicy`)
* **metadata.namespace:** the name of your namespace (this example uses `mysql`)
* **policy.daily.time:** the time of the day when Portworx will trigger the operation (this example triggers the operation every day at 10:14 PM)

    ```text
    apiVersion: stork.libopenstorage.org/v1alpha1
    kind: SchedulePolicy
    metadata:
      name: testpolicy
      namespace: mysql
    policy:
      daily:
        time: "10:14PM"
    ```

### Weekly SchedulePolicy

A weekly `SchedulePolicy` includes the following fields and values:

* **apiVersion:** the version of the Stork scheduler (this example uses `stork.libopenstorage.org/v1alpha1`)
* **kind:** as `SchedulePolicy`
* **metadata.name:** the name of the `SchedulePolicy` object (this example uses `testpolicy`)
* **metadata.namespace:** the name of your namespace (this example uses `mysql`)
* **policy.weekly.day:** the day of the week when Portworx will trigger the operation (this example triggers the operation every Thursday)
* **policy.weekly.time:** the time of the day when Portworx will trigger the operation (this example triggers the operation at 10:13 PM)

    ```text
    apiVersion: stork.libopenstorage.org/v1alpha1
    kind: SchedulePolicy
    metadata:
      name: testpolicy
      namespace: mysql
    policy:
      weekly:
        day: "Thursday"
        time: "10:13PM"
    ```

### Monthly SchedulePolicy

A monthly `SchedulePolicy` includes the following fields and values:

* **apiVersion:** the version of the Stork scheduler (this example uses `stork.libopenstorage.org/v1alpha1`)
* **kind:** as `SchedulePolicy`
* **metadata.name:** the name of the `SchedulePolicy` object (this example uses `testpolicy`)
* **metadata.namespace:** the name of your namespace (this example uses `mysql`)
* **policy.monthly.day:** the day of the month when Portworx will trigger the operation (this example triggers the operation on the 14th of every month)
* **policy.monthly.time:** the time of the day when Portworx will trigger the operation (this example triggers the operation at 8:05 PM)

    ```text
    apiVersion: stork.libopenstorage.org/v1alpha1
    kind: SchedulePolicy
    metadata:
      name: testpolicy
      namespace: mysql
    policy:
      monthly:
        date: 14
        time: "8:05PM"
    ```
