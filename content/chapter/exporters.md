---
title: "Exporters"
description: "Expose metrics for Prometheus"
repo: ""
tags: []
weight: 0
date: 97
draft: false
---

## node_exporter

The node exporter enables basic monitoring of linux machines (and other unix
like systems.)

1. Download and run the node_exporter
1. Enable the systemd collector (might require to run as root)
1. Add your node_exporter and your neighbors to prometheus

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


## JMX exporter

1. Download [Jenkins](https://jenkins.io/download/)
1. Download the [JMX
   exporter](https://github.com/prometheus/jmx_exporter)
1. Run Jenkins with the JMX exporter and add it to Prometheus

{{% hidden "solution" %}}
```
java -javaagent:./jmx_prometheus_javaagent-0.3.1.jar=8081:config.yml -jar jenkins.war
```
{{% /hidden %}}

{{% hidden "config.yml" %}}
```
---
startDelaySeconds: 0
```
{{% /hidden %}}


1. *exercise* Create a dashboard with:
    - JVM version
    - Uptime
    - Threads
    - Heap Size
    - Memory Pool size

## Grok exporter

- Download [grok exporter](https://github.com/fstab/grok_exporter)
- Create a simple job in Jenkins
- Re run Jenkins to output to a file (add `&> jenkins.log`)

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


## Blackbox exporter

*Exercise*

- Monitor the OSMC website (DNS + HTTP) using the blackbox exporter
- Check with prometheus blackbox exporter when the SSL certificate will expire
  in days
- Create a dashboard with the detailed time it takes to get the OSMC website.
