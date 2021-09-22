---
title: "Exporters"
description: "Expose metrics for Prometheus"
repo: ""
tags: []
weight: 0
date: 97
draft: false
---


Exporters are HTTP servers that expose metrics. They can translate Prometheus
queries into domain-specific queries. They then turn the results into Prometheus
metrics.

There are hundreds of known exporters, most of them coming from the community. A
few exporters are maintained by the Prometheus team.

## node_exporter

The node exporter enables basic monitoring of linux machines (and other unix
like systems).

1. Download the [node_exporter](https://prometheus.io/download/) {{% version "node_exporter" %}}.
1. Extract it

    ```shell
    $ tar xvf Downloads/node_exporter-{{% version "node_exporter" %}}.linux-amd64.tar.gz
    ```

1. List the files

    ```shell
    $ ls node_exporter-{{% version "node_exporter" %}}.linux-amd64
    ```

1. Launch the node_exporter

    ```shell
    $ cd node_exporter-{{% version "node_exporter" %}}.linux-amd64
    $ ./node_exporter
    ```

1. Open your browser at [http://127.0.0.1:9100](http://127.0.0.1:9100)
1. Add your node_exporter and your neighbors to prometheus.

### collectors

The Node Exporter has multiple collectors, some of them disabled by default.

*Exercise*

1. Enable the systemd collector

### textfile collector

*Exercise*

Move the metrics created before (company name, random number..) on port 5678 to
be collected by the node exporter.

Do you see use cases for this feature?

### Dashboards

*Exercise*

Create two dashboards: a dashboard that will show the network bandwidth of a
server by interface, and a dashboard that will show the disk space available per
disk.

{{% tip %}}
You can use `{job="node_exporter"}` in prometheus to see the metrics, or you
can directly open the /metrics of the node_exporter in your browser.
{{% /tip %}}

---

## JMX exporter


The JMX exporter is useful to monitor Java applications. It can be loaded as a
"side car" (Java agent), in the same JVM's as the applications.

1. Download [Jenkins](https://jenkins.io/download/)
1. Download the [JMX
   exporter](https://github.com/prometheus/jmx_exporter) {{% version "jmx_exporter" %}}.
1. Run Jenkins with the JMX exporter and add it to Prometheus

{{% hidden "solution" %}}
```shell
$ java -javaagent:./jmx_prometheus_javaagent-{{% version "jmx_exporter" %}}.jar=8081:config.yml -jar jenkins.war
```
{{% /hidden %}}

{{% hidden "config.yml" %}}
```yaml
---
startDelaySeconds: 10
```
{{% /hidden %}}

*exercise*

1. Create a dashboard with:
    - JVM version
    - Uptime
    - Threads
    - Heap Size
    - Memory Pool size

---

## Grok exporter

1. Download [grok exporter](https://github.com/fstab/grok_exporter) {{% version "grok_exporter" %}}
1. Extract it

    ```shell
    $ unzip Downloads/grok_exporter-{{% version "grok_exporter" %}}.linux-amd64.zip
    ```

1. List the files

    ```shell
    $ ls grok_exporter-{{% version "grok_exporter" %}}.linux-amd64
    ```

1. Create a simple job in Jenkins
1. Re run Jenkins to output to a file (add `&> jenkins.log`)

*exercise*

- Create a job with name "test" and command "sleep 10"
- Run the job and look for "INFO: test #2 main build action completed: SUCCESS"
  in the logs
- Create a counter and a gauge for those: `job_last_build_number` and
  `job_build_total`. The name of the job should be a label, and for the
  `job_build_total` the status should too be a label.

{{% hidden "solution" %}}
```
global:
    config_version: 2
input:
    type: file
    path: ../jenkins.log
    readall: true
grok:
    patterns_dir: ./patterns
metrics:
    - type: counter
      name: job_build_total
      help: Counter for the job runs.
      match: 'INFO: %{WORD:jobname} #%{NUMBER:jobid} main build action
completed: %{WORD:status}'
      labels:
          jobname: '{{.jobname}}'
          status: '{{.status}}'
    - type: gauge
      name: job_last_build_number
      help: Number of the last build
      match: 'INFO: %{WORD:jobname} #%{NUMBER:jobid} main build action
completed: %{WORD:status}'
      labels:
        jobname: '{{.jobname}}'
      value: '{{.jobid}}'
server:
    port: 9144
```
{{% /hidden %}}

---


## Blackbox exporter

1. Download the [blackbox_exporter](https://prometheus.io/download/) {{% version "blackbox_exporter" %}}.
1. Extract it

    ```shell
    $ tar xvf Downloads/blackbox_exporter-{{% version "blackbox_exporter" %}}.linux-amd64.tar.gz
    ```

1. List the files

    ```shell
    $ ls blackbox_exporter-{{% version "blackbox_exporter" %}}.linux-amd64
    ```

1. Launch the blackbox_exporter

    ```shell
    $ cd blackbox_exporter-{{% version "blackbox_exporter" %}}.linux-amd64
    $ ./blackbox_exporter
    ```

1. Open your browser at [http://127.0.0.1:9115](http://127.0.0.1:9115)
1. Add your blackbox_exporter and your neighbors to prometheus

*Exercise*

- Monitor the Inuits website (DNS + HTTP) using the blackbox exporter
- Check with prometheus blackbox exporter when the SSL certificate will expire
  in days
- Create a dashboard with the detailed time it takes to get the OSMC website.

