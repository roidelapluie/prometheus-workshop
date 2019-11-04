---
title: "Config files"
description: "Some configuration files for the workshop"
repo: ""
tags: []
weight: 0
date: 10
draft: false
---

## prometheus.yml

```
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 192.168.26.1:9093
      - 192.168.26.2:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __address__
        replacement: "${1}:9090"
    file_sd_configs:
      - files:
        - workshop.yml
        refresh_interval: 10s
  - job_name: 'node'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __address__
        replacement: "${1}:9100"
    file_sd_configs:
      - files:
        - workshop.yml
        refresh_interval: 10s
  - job_name: 'grafana'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __address__
        replacement: "${1}:3000"
    file_sd_configs:
      - files:
        - workshop.yml
        refresh_interval: 10s
  - job_name: 'federation'
    metrics_path: /federate
    params:
        "match[]": [up]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __address__
        replacement: "${1}:9090"
    metric_relabel_configs:
      - source_labels: [__name__]
        target_label: __name__
        regex: up
        replacement: federate_up

    file_sd_configs:
      - files:
        - workshop.yml
        refresh_interval: 10s
  - job_name: 'jenkins'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __address__
        replacement: "${1}:8081"
    file_sd_configs:
      - files:
        - workshop.yml
        refresh_interval: 10s
  - job_name: 'aletmanager'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __address__
        replacement: "${1}:9093"
    file_sd_configs:
      - files:
        - workshop.yml
        refresh_interval: 10s
  - job_name: 'grok'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __address__
        replacement: "${1}:9144"
    file_sd_configs:
      - files:
        - workshop.yml
        refresh_interval: 10s
```

## workshop.yml

```
- targets: ['192.168.26.1']
  labels:
      name: me
- targets:
  - '192.168.26.3'
  - '192.168.26.4'
  - '192.168.26.5'
  - '192.168.26.6'
  - '192.168.26.7'
  labels:
      name: right
- targets:
  - '192.168.26.8'
  - '192.168.26.9'
  - '192.168.26.10'
  - '192.168.26.11'
  - '192.168.26.12'
  - '192.168.26.13'
  - '192.168.26.14'
  - '192.168.26.15'
  labels:
      name: left
```

## first_rules.yml

```
groups:
  - name: example
    rules:
    - alert: a target is down
      for: 5m
      expr: up == 0
      labels:
        priority: high
      annotations:
        text: "{{$labels.job}} is down!"
```

## grok_exporter.yml

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

## alertmanager.yml

```
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
  routes:
    - match:
        priority: high
        continue: true
        receiver: 'sms.hook'
receivers:
- name: 'sms.hook'
  webhook_configs:
  - url: 'https://webhook.site/5c702a0d-2c02-4f70-a8ee-9ac45d2ce2b9'
- name: 'web.hook'
  webhook_configs:
  - url: 'https://webhook.site/5c702a0d-2c02-4f70-a8ee-9ac45d2ce2b9'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```
