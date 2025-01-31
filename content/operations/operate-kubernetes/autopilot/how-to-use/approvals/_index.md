---
title: "Action approvals"
linkTitle: "Action approvals"
keywords: autopilot
hidesections: true
description: Instructions on how to enable approvals for actions resulting from autopilot rules
weight: 30000
aliases:
    - /portworx-install-with-kubernetes/autopilot/how-to-use/approvals/
---
## Prerequisites

* Autopilot 1.3.0 and above

## Overview 

When you create an AutopilotRule in your cluster, Autopilot will execute actions on the objects in the cluster when the conditions in the AutopilotRule are met. An example action would be resize of a PVC when its usage increases.

The diagram below captures this workflow. Autopilot monitors the cluster, predicts actions that need to be taken based on the rules you have created and acts upon them by executing the actions.

<img src="/img/aut-workflow-no-approval.png" alt="drawing" width="400" height="400"/>

Autopilot allows you to approve actions that result from an AutopilotRule before it executes them. The workflow remains similar to above with the addition of the **approve** stage. Approvals are not required by default.

<img src="/img/aut-workflow-approval.png" alt="drawing" width="400" height="400"/>


## Workflow

To enable Autopilot action approvals in your cluster, following one of the following guides.

{{<homelist series="aut-approval-walkthroughs">}}
 
## Known issues

Following are the known issues in Autopilot 1.3.0 related to Action approvals.

#### Updating an AutopilotRule to not require approvals

* **Issue**: If an AutopilotRule has `enforcement: approvalRequired` and is later updated to remove the `enforcement`, existing approvals that resulted from that rule will not be cancelled. 
* **Workaround**:  Delete the AutopilotRule and recreate it without the `enforcement`. This will cancel existing approvals and no subsequent approvals will be generated for that AutopilotRule.
