---
title: "Prometheus"
description: "Install & Setup Prometheus"
repo: ""
tags: []
weight: 0
date: 100
draft: false
---

Prometheus is an open source monitoring system designed around metrics. It is
a large ecosystem, with plenty of different components.

The [prometheus documentation](https://prometheus.io/docs/introduction/overview/) provides an overview of those components:

![Prometheus Architecture, CC-BY-SA 4.0, from the Prometheus Authors 2014-2019](https://prometheus.io/assets/architecture.png)


## How Prometheus works

Prometheus monitoring is based on metrics, exposed on HTTP endpoints. The
Prometheus server is "active" and starts the polling. That polling (called
"scraping") happens at a high interval (usually 15s or 30s).

Each monitored target must expose a metrics endpoint. That endpoint exposes
metrics in the Prometheus HTTP format or in the OpenMetrics format.

Once collected, those metrics are mutated by Prometheus, which adds an instance
and job label. Optionally, extra relabeling configured by the user occurs.

## The Prometheus server

1. Download the [prometheus server](https://prometheus.io/download/) {{% version "prometheus" %}}.
1. Extract it

    ```shell
    $ tar xvf Downloads/prometheus-{{% version "prometheus" %}}.linux-amd64.tar.gz
    ```

1. List the files

    ```shell
    $ ls prometheus-{{% version "prometheus" %}}.linux-amd64
    ```

1. Launch prometheus

    ```shell
    $ cd prometheus-{{% version "prometheus" %}}.linux-amd64
    $ ./prometheus
    ```

1. Open your browser at [http://127.0.0.1:9090](http://127.0.0.1:9090)

1. Look at the TSDB data

{{% note title="tsdb" %}}
Prometheus stores its data in a database called
[tsdb](https://github.com/prometheus/prometheus/tree/master/tsdb). The TSDB is
self-maintained by the server, which manages the data lifecycle.
{{% /note %}}


---

## The web ui

There is a lot of information that can be found in the prometheus server web ui.

Try to find:

- The version of prometheus
- The duration of data retention
- The "targets" that are scraped by default
- The "scrape" interval

{{% note title="React UI" %}}
The Prometheus UI went under a huge refactoring in 2020. It is now react-based,
with powerful autocomplete features. There is still a link to access the
"classic" UI.
{{% /note %}}

---

## promtool

promtool is a command line tool provided with Prometheus.

With promtool you can:

- Validate Prometheus configuration

   ```shell
   $ ./promtool check config prometheus.yml
   ```

- Query Prometheus

  ```shell
  $ ./promtool query instant http://127.0.0.1:9090 up
  ```

{{% info %}}
The `up` metric is added by prometheus on each scrape. Its value is 1 if the
scrape has succeeded, 0 otherwise.
{{% /info %}}

- Create blocks from OpenMetrics files or recording rules, aka backfill.

---

## Adding targets

{{% note %}}
At this point, make sure you understand the basis of [YAML](https://yaml.org).
{{% /note %}}

*exercise*

- Open prometheus.yml
- Add each one's prometheus server as targets to your prometheus server.
- Look the status (using up or the target page)


What is a job? What is an instance?

{{% tip %}}
You do not need to reload prometheus: you can just send a `SIGHUP` signal to
reload the configuration:

```shell
$ killall -HUP prometheus
```
{{% /tip %}}

---

## Admin commands

1. Enable admin commmands

    ```shell
    $ ./prometheus --web.enable-admin-api
    ```

1. Take a snapshot

    ```shell
    $ curl -XPOST http://localhost:9090/api/v1/admin/tsdb/snapshot
    ```

    Look in the data directory.


    {{% note %}}
    This is snapshotting the TSDB. There is another kind of snapshot, Memory
    Snapshot on Shutdown, which is a different feature.
    {{% /note %}}


1. Delete a timeserie

    ```shell
    $ curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]=process_start_time_seconds{job="prometheus"}'
    ```

---

## Federation

### File_sd

Now, let's move to file_sd.

Create a file:

```yaml
- targets:
    - 127.0.0.1
  labels:
    name: Julien
- targets:
    - 127.0.0.2
  labels:
    name: John
```

With your IP + your neighbors.

Name it `users.yml`.

Adapt Prometheus configuration:

```yaml
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    file_sd_configs:
      - files:
         - users.yml
    relabel_configs:
      - source_labels: [__address__]
        target_label: __address__
        replacement: "${1}:9090"
```


Duplicate the job, but with the following instructions:

- The new job should be called "federation"
- The new job should query http://127.0.0.1:9090/federate?match[]=up
- The "up" metric fetched should be renamed to external_up

{{% tip %}}
The name of a metric is a label too! It is the `__name__` label.
{{% /tip %}}

{{% hidden "Solution" %}}

```yaml
- job_name: 'federation'
  metrics_path: '/federate'
  params:
    'match[]':
      - up
  file_sd_configs:
    - files:
       - users.yml
  relabel_configs:
    - source_labels: [__address__]
      target_label: __address__
      replacement: "${1}:9090"
  metric_relabel_configs:
    - source_labels: [__name__]
      target_label: __name__
      regex: up
      replacement: federate_up
```

{{% /hidden %}}

### DigitalOcean SD

Now, let's move to digitalocean_sd.

In your VM, there is a /etc/do_read file with a digitalocean token.

The version of Prometheus you have has native integration with DigitalOcean.

Adapt Prometheus configuration:

```yaml
  - job_name: 'prometheus'
    digitalocean_sd_configs:
      - bearer_token_file: /etc/do_read
        port: 9090
    relabel_configs:
      - source_labels: [__meta_digitalocean_tags]
        regex: '.*,prometheus_workshop,.*'
        action: keep
```

Reload Prometheus:

```shell
killall -HUP prometheus
```

You should see the 10 prometheus servers.

Duplicate the job, but with the following instructions:

- The new job should be called "federation"
- The new job should query http://127.0.0.1:9090/federate?match[]=up
- The "up" metric fetched should be renamed to external_up

{{% tip %}}
The name of a metric is a label too! It is the `__name__` label.
{{% /tip %}}

{{% hidden "Solution" %}}

```yaml
- job_name: 'prometheus'
  digitalocean_sd_configs:
    - bearer_token_file: /etc/do_read
  relabel_configs:
  - source_labels: [__meta_digitalocean_tags]
    regex: '.*,prometheus_workshop,.*'
    action: keep
  - source_labels: [__meta_digitalocean_droplet_name]
    target_label: instance
  - source_labels: [__meta_digitalocean_public_ipv4]
    target_label: __address__
    replacement: '$1:9090'
- job_name: 'federation'
  metrics_path: '/federate'
  digitalocean_sd_configs:
    - bearer_token_file: /etc/do_read
  params:
    'match[]':
      - up
  relabel_configs:
  - source_labels: [__meta_digitalocean_tags]
    regex: '.*,prometheus_workshop,.*'
    action: keep
  - source_labels: [__meta_digitalocean_droplet_name]
    target_label: instance
  - source_labels: [__meta_digitalocean_public_ipv4]
    target_label: __address__
    replacement: '$1:9090'
  metric_relabel_configs:
    - source_labels: [__name__]
      target_label: __name__
      regex: up
      replacement: federate_up
```

{{% /hidden %}}

## Last exercise

Prometheus fetches Metrics over HTTP.

Metrics have a name and labels.

As an exercise, let's build on top of our previous example:

In a new directory, create a file called "metrics"

Add some metrics:

```
company{name="inuits"} 1
favorite_color{name="red"} 1
random_number 10
workshop_step 1
```

then, run `python -m SimpleHTTPServer 5678` and add it to prometheus (and your
neighbors too).
