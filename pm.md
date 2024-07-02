## SUMMARY
This postmortem analyzes recurrent Kubernetes pod restart issues, initiation failures, and sluggish performance after cluster downsizing and reconfiguration.

## SYMPTOMS
- Pods constantly restarting.
- Pod initiation failures.
- Large number of pods in ContainerStatusUnknown.
- Sluggish performance and timeout issues.

## IMPACT
Users experienced significant delays and interruptions due to frequent pod restarts and performance degradation, leading to reduced system reliability. The inability of some applications to start compounded these disruptions, affecting overall productivity. This instability prolonged resolution times and reduced trust in the infrastructure's reliability.

## TIMELINE

| **Date**       | **Time** (UTC) | **Event Description**                            |
|----------------|----------------|--------------------------------------------------|
| 06/24/2024     | 08:00          | Pods began restarting frequently.                |
| 06/24/2024     | 08:15          | Initiation issues for new pod deployments.       |
| 06/24/2024     | 10:00          | Noted many pods in ContainerStatusUnknown status.|
| 06/24/2024     | 11:30          | Performance degradation and timeout issues observed.|
| 06/25/2024     | 09:00          | Investigation pinpointed Verticalpodautoscaler and cluster downsizing as potential causes.|
| 06/26/2024     | 14:00          | Applied configuration changes to address issues. |
| 06/27/2024     | 08:00          | Monitoring indicated stabilization.              |

## ROOT CAUSE(S) AND RESOLUTION
- **Root Cause:** Inadequate memory and CPU requests due to downsizing.
  - **Resolution:** Adjusted resource requests and limits.
- **Root Cause:** Misalignment of some applications with Verticalpodautoscaler.
  - **Resolution:** Reverted Verticalpodautoscaler settings for incompatible applications.
- **Root Cause:** Configuration changes causing unexpected behavior.
  - **Resolution:** Rolled back recent configuration changes.

## ACTION ITEMS
- Review and adjust resource allocation policies.
- Identify and update applications incompatible with Verticalpodautoscaler.
- Implement improved monitoring for future configuration changes.
- Conduct thorough testing before applying cluster downsizing or significant updates.
- Enhance team training on Kubernetes best practices to prevent recurrence.


## Learnings

Implementing numerous changes simultaneously can lead to instabilities that are difficult to diagnose.
