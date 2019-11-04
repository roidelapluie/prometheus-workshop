---
title: "Alerting"
description: "Doing something with those metrics"
repo: ""
tags: []
weight: 0
date: 90
draft: false
---

## Recording rules and alerts

{{% note %}}
Prometheus is using the [Go Templating System](https://prometheus.io/docs/prometheus/latest/configuration/template_examples/)
for alerting, in both Prometheus and Alertmanager.
{{% /note %}}

Prometheus splits the alerting role in 3 components:

- prometheus server which will calculate the alerts
- alertmanager which will dispatch the alerts
- webhook receivers that will handle the alerts


{{% note %}}
Alerts and recording rules are close to each other. They are queries that are
run at regular interval by prometheus and write new metrics into the tsdb.
{{% /note %}}

*Exercise*

Create, in Prometheus, an alert when a target is down.

*Exercise*

Create, in Prometheus, an alert when a grafana server is down, with an extra label:
priority=high.

*Exercise*

Create a **recording rule** to get the % of disk space used
and alert on > 50% of disk space used.


What is the difference between recording and alerting?

What is an annotation?

What is a "group" of recording rules?

How to see the rules and the alerts in the UI?

What is a pending alert?

Bonus: Alerts unit test (if there is enough time)

{{% tip %}}
Prometheus generates an ALERTS metric with the active/pending alerts.
{{% /tip %}}

## Alertmanager

1. Download the [alertmanager](https://prometheus.io/download/) {{% version "alertmanager" %}}.
1. Extract it

    ```shell
    $ tar xvf Downloads/alertmanager-{{% version "alertmanager" %}}.linux-amd64.tar.gz
    ```

1. List the files

    ```shell
    $ ls alertmanager-{{% version "alertmanager" %}}.linux-amd64
    ```

1. Launch the alertmanager

    ```shell
    $ cd alertmanager-{{% version "alertmanager" %}}.linux-amd64
    $ ./alertmanager
    ```
1. Open your browser at [http://127.0.0.1:9093](http://127.0.0.1:9093)
1. Add your alertmanager and your neighbors to prometheus
1. Connect Prometheus and Alertmanager together
1. Look for the alerts coming.

- What are the 4 roles of alertmanager?
- What are the different timers in alertmanager?

*Exercise*

Use https://webhook.site/ to get a webhook URL.

Send alerts to that https://webhook.site/ URL.

For the priority=high alerts, send an email instead of a webhook.


- Can you explain the HA model of prometheus?
- How can I send an alert to multiple targets?

*Exercise*

How can you check that two alertmanager config are in sync?

{{% note %}}
There is a `alertmanager_config_hash` metric
{{% /note %}}

{{% hidden "Solution" %}}
```
count(count_values("config", alertmanager_config_hash)) != 1
```
{{% /hidden %}}

*Exercise*

Make a big cluster of alert managers

### Amtool

Amtool is the CLI tool for alertmanager

You can use it to e.g. create silences.

```shell
$ ./amtool silence --alertmanager.url=http://127.0.0.1:9093 add job=grafana priority=high -d 15m -c "we redeploy grafana" -a Julien
```

That will return the UID of the silence that you can use to expire it.

### Karma

[karma](https://github.com/prymitive/karma) is a dashboard for alertmanager
