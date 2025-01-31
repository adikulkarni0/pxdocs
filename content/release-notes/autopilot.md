---
layout: page
title: "Autopilot Release Notes"
description: Release notes for the Portworx Autopilot component.
keywords: portworx, release notes, components
weight: 35000
series: release-notes
aliases:
    - /reference/release-notes/autopilot
---

## 1.3.2

December 14, 2021

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
| PD-1047 | Autopilot 1.3.2 now supports installations that use PX-Security. Follow the instructions in [Autopilot Install and Setup](/operations/operate-kubernetes/autopilot/how-to-use/install-autopilot/#autopilot-with-px-security) to use Autopilot with PX-Security. |

## 1.3.1

November 22, 2021

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
| PD-1035 | Autopilot 1.3.1 now supports Kubernetes 1.22 and higher |

## 1.3.0

Sep 02, 2020

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Number**|**Improvement Description**|
|----|----|
|AUT-113|Added support to **rebalance Portworx storage pools**. Refer to [documentation](/operations/operate-kubernetes/autopilot/use-cases/rebalance-pool/) for more details.|
|AUT-136|Added support to users to require **approval** before actions are taken for a AutopilotRule. Refer to [documentation](/operations/operate-kubernetes/autopilot/how-to-use/approvals/) for more details.|
|AUT-138|Added support for `requiredMatches` in the `AutopilotRule` CRD. This allows users to specify number of conditions that are required to match in the rule. See the [Automatically rebalance Portworx storage pools](/operations/operate-kubernetes/autopilot/use-cases/rebalance-pool/) use case on example on how this can be used.|
|AUT-144|Added support for Autopilot to create **Github Pull Requests** to approvals for its actions. This enables users to integrate **GitOps** workflows with Autopilot. Refer to [documentation](/operations/operate-kubernetes/autopilot/how-to-use/approvals/walkthrough-github/) for more details.|
|AUT-157|Make AutopilotRuleObject namespace scoped. Refer to [documentation](/operations/operate-kubernetes/autopilot/how-to-use/operate-and-troubleshoot/#get-recent-statuses-using-autopilotruleobjects) for usage of AutopilotRuleObjects.|
|AUT-205|[Improve support bundle collection](/operations/operate-kubernetes/autopilot/how-to-use/operate-and-troubleshoot/#collecting-a-support-bundle)|

The following issues have been fixed:

|**Issue Number**|**Issue Description**|
|----|----|
|AUT-83|Autopilot pool expand should never bring Portworx out of quorum<br/><br/>**User impact:** In certain situations where 2 or more pools have non-intersection volumes, Autopilot can triggered expand on multiple pools at the same time which can bring the PX cluster out of quorum.<br/><br/>**Resolution:** Autopilot will now perform expansion on only one pool at a given time. Subsequent pools will have their actions in pending state until the previous one is complete.|
|AUT-87|If an action is declined, Autopilot will now perform exponential backoff before retrying it.<br/><br/>**User impact:** If an action was declined due to the maxsize being hit in the AutopilotRule for a PVC resize, Autopilot used to aggressively retry flooding logs and events.<br/><br/>**Resolution:** Autopilot will now perform exponential backoff.|
|AUT-169|Autopilot loses track of volume for rule when PX-Backup backs up and restores to same namespace.<br/><br/>**User impact:** If a user has a PVC that was restored from a backup or pre-provisioned, Autopilot would not be able to track the metrics of the PVC correctly.<br/><br/>**Resolution:** Autopilot was incorrect assuming the volume name of a PVC by using the PVC UUID. Instead now, it will use the actual volume name from the PVC spec.|
|AUT-209|Fixed an issue where Autopilot incorrectly determines that conditions are not met for an object.<br/><br/>**User impact:** There are situations where the prometheus API reports empty metrics when it is initializing and it hasn't scrapped its targets. If Autopilot queries prometheus during such times, it will find empty metrics and will be misled to consider the conditions as not being met for that object. <br/><br/>**Resolution:** Autopilot will no longer accept metrics from Prometheus if a Portworx metrics target is not healthy. The Portworx metrics target will become healthy when Prometheus has initialized completely and has been able to perform a successful scrape.|

## 1.2.1

May 30, 2020

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Description**|
|----|
| The [PVC resize action](/operations/operate-kubernetes/autopilot/reference/#openstorage-io-action-volume-resize) no longer requires a Portworx Autopilot Capacity Management license |

## 1.2.0

March 20, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Description**|
|----|
| Added support for a new parameter, `scalesize`, under the storage pool expand action that allows you to increase storage pool capacity by a fixed amount. See details on all action parameters [here](/operations/operate-kubernetes/autopilot/reference/#openstorage-io-action-storagepool-expand). |
| Added validation for the AutopilotRule CRD |
| Added an alert event that occurs when Autopilot detects an action that cannot be performed due to license restrictions. See the [list of alerts](/operations/operate-kubernetes/autopilot/reference/#autopilot-events) for more information. |

## 1.1.0

February 19, 2020

### Improvements

Portworx has upgraded or enhanced functionality in the following areas:

|**Improvement Description**|
|----|
| Added a new CRD, called AutopilotRuleObject, which can be used to check for useful events in objects that autopilot monitors, such as PVCs and StoragePools. |
| Added basic metrics for monitoring Autopilot and Grafana dashboards to view them. To view Autopilot metrics, follow the steps in the [Prometheus and Grafana](/install-portworx/monitoring) article. |

### Fixes

The following issues have been fixed:

|**Issue Description**|
|----|
| When an Autopilot pod restarted while expanding a storage pool, it sometimes started expanding another storage pool.<br/><br/>**User impact:** If multiple pools for a volume started expansion together, the volume could go out of quorum.<br/><br/>**Resolution:** Autopilot pods now correctly wait for previous storage pool expansions to complete when they're restarted.|
| Deleting Autopilot rules or PVCs sometimes caused the Autopilot pod to crash<br/><br>**User impact:** If Autopilot crashed, it could start expanding another storage pool while one was already in being expanded.<br/><br/>**Resolution:** The Autopilot pod no longer crashes when rules or PVCs are deleted.|

## 1.0.0

November 18, 2019

Introducing Portworx Autopilot! See the [Autopilot](https://2.3.docs.portworx.com/portworx-install-with-kubernetes/autopilot/) section of the documentation for more information
