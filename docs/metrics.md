# Metrics Monitoring

Metrics monitoring is different because it does not assume that a situation can
be explained in fixed states. Instead, it brings you inside your system and
provides dizens of metrics that you can then analyze and understand.

Even if you do not need all the metrics, it is better to collect them to look
further what they look like.

## What is a metrics

- Name
- Timestamp
- Labels
- Value
- Type

*What are the 4 types of metrics prometheus knows?*

- Gauge
- Counters
- Histograms
- Summaries

The number of http requests is expressed in ...

The duration in ...

The number of active sessions is a ...


What are the labels added by prometheus ?

What are the labels prometheus knows but does not add?

## Labels matching

For prometheus to do calculations and comparisons of metrics, labels must match
one each side (except name)

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


## Aggregation

How can I get the sum of scraped metrics by job (2 ways)? -- exclusion and
inclusion.

How can I get the % of /federate over the other
prometheus_http_request_duration_seconds_count ?

??? success "Solution"
    ```
    rate(prometheus_http_request_duration_seconds_count{handler="/federate"}[5m])
    / ignoring(handler) group_right sum without (handler)
    (rate(prometheus_http_request_duration_seconds_count[5m]))
    ```

## Over Time

What is the difference between max and max_over_time?

## Max, Min, Bottomk, Topk

What is the difference between max and topk?

## Time functions

day()
day_of_week()

How to use the optional argument of day_of_week?

What is the timestamp() function? How can it be useful?


!!! tip
    You can use Grafana Explore feature to get help and autocomplete on
    Prometheus (currently still a "beta" feature).

## And/Or

Can you think of any usecases for and/or/unless?
