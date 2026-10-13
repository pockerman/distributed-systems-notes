# Lesson 69: SLIs, SLOs and Error Budgets

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

We cannot make a system 100% reliable, and trying to would stop us shipping. Service level objectives define how reliable is reliable enough, and error budgets turn that into a decision rule.

## Outline

- SLIs, SLOs and SLAs
- Choosing good SLIs: availability, latency percentiles, freshness
- Error budgets and burn rates
- Alerting on symptoms, not causes
- Multi-window, multi-burn-rate alerts
- Availability of composed services (serial and parallel dependencies)

## Related lessons

- [Lesson 68: Observability: Logs, Metrics and Traces](observability.md)
- [Lesson 31: Failure Causes](../2026-01-28-Failure-Causes.md)

## References

1. Betsy Beyer, Chris Jones, Jennifer Petoff and Niall Richard Murphy (eds.), _Site Reliability Engineering_, O'Reilly Media
2. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
3. Alex Hidalgo, _Implementing Service Level Objectives_, O'Reilly Media
