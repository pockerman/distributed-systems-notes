# Lesson 67: Autoscaling

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

Load changes through the day, and paying for peak capacity all the time is wasteful. Autoscaling adds and removes capacity as load changes.

## Outline

- Horizontal vs vertical autoscaling
- Reactive scaling: metrics, thresholds and cooldowns
- Predictive and scheduled scaling
- Scaling lag, cold starts and warm pools
- Using Little's Law to size a fleet
- Autoscaling in Kubernetes (HPA, VPA, cluster autoscaler)

## Related lessons

- [Lesson 13: Little's Law](../2026-05-18-Scalability-Little-Law.md)
- [Lesson 14: Cold & Warm Starts](../2026-05-20-Scalability-Cold-Warm-Starts.md)
- [Lesson 33: Upstream Resiliency](../2026-02-09-Upstream-Resiliency.md)

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
2. Betsy Beyer, Chris Jones, Jennifer Petoff and Niall Richard Murphy (eds.), _Site Reliability Engineering_, O'Reilly Media
