---
title: "Metrics Monitoring"
description: "Querying Prometheus"
repo: ""
tags: []
weight: 0
date: 99
draft: false
---

Metrics monitoring is different because it does not assume that a situation can
be explained in fixed states. Instead, it brings you inside your system and
provides dozens of metrics that you can then analyze and understand.

Even if you do not need all the metrics, it is better to collect them to look
further what they look like.

## What is a metric

- **Name**
- Timestamp
- Labels
- **Value**
- Type

### Labels

Labels are used to add metadata to metrics. That can be used to differenciate
then, e.g. by adding a status code, a URI handler, a function name, ...

### Types of metrics

Gauge
: Metric that can go up and down

Counters
: Metric that can only go up and starts at 0

Histograms
: Metrics that put data into buckets. An histogram is composed of 3 metrics:
sum, buckets and count.

Summaries
: Metrics that calculate quantiles over observed values. A summary is composed
of 3 metrics: sum, quantiles and count.

[Upstream documentation](https://prometheus.io/docs/concepts/metric_types/)

### Exercises

The number of http requests is expressed in ...

The duration in ...

The number of active sessions is a ...

What are the labels added by prometheus ?

What are the labels prometheus knows but does not add?

---

## Labels matching

For prometheus to do calculations and comparisons of metrics, labels must match
one each side (except `__name__`, the name of the metric)

---

## Functions

Some important functions:

- Math: = - / *

- rate
- sum
- deriv
- delta
- increase
- count

In the list above, which ones should be used with counters, and which ones with
gauges?


What is the difference between irate and rate? idelta and delta?

---

## Aggregation

How can I get the sum of scraped metrics by job (2 ways)? -- exclusion and
inclusion.

{{% tip %}}
The `scrape_samples_scraped` metric is added by prometheus after sraping a
target and indicates the number of metrics for that job at that scrape.
{{% /tip %}}

{{% hidden "Solution" %}}
Using by:
```
sum(scrape_samples_scraped) by (job)
```

Using without:
```
sum(scrape_samples_scraped) without (instance)
```
{{% /hidden %}}

How can I get the % of `handler="/federate"` over the other
`prometheus_http_request_duration_seconds_count` ?

{{% hidden "Solution" %}}
```
100 *
rate(prometheus_http_request_duration_seconds_count{handler="/federate"}[5m])
/ ignoring(handler) group_right sum without (handler)
(rate(prometheus_http_request_duration_seconds_count[5m]))
```
{{% /hidden %}}

---

## Over Time

What is the difference between `max` and `max_over_time`?

---

## Max, Min, Bottomk, Topk

What is the difference between `max(x)` and `topk(1, x)`?

---

## Time functions

`day()`

`day_of_week()`

How to use the optional argument of `day_of_week`?

What is the `timestamp()` function? How can it be useful?

{{% tip %}}
You can use Grafana Explore feature to get help and autocomplete on
Prometheus (currently still a "beta" feature). That feature will likely
come in the next release of Prometheus!
{{% /tip %}}

## And/Or

Can you think of any usecases for `and`/`or`/`unless`?
