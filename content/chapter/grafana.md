---
title: "Grafana"
description: "Create beautiful dashboards"
repo: ""
tags: []
weight: 0
date: 98
draft: false
---

## Download and run grafana

1. Go to the [Grafana website](https://grafana.com/) and download the latest stable release
1. For this exercise we will use the Standalone Linux Binaries(64 Bit)
  ```shell
  $ wget https://dl.grafana.com/oss/release/grafana-{{% version "grafana" %}}.linux-amd64.tar.gz
  $ tar -zxvf grafana-{{% version "grafana" %}}.linux-amd64.tar.gz
  $ cd grafana-{{% version "grafana" %}}
  $ ./bin/grafana-server
  ```
1. Open Grafana in your browser: http://127.0.0.1:3000 (username: admin;
   password: admin)


## Setup prometheus

Add your prometheus server as a datasource + import the prometheus dashboards.

Monitor grafana in prometheus (add it as a target).

Look at the grafana dashboards.


## Create a new dashboard

Create a new dashboard which enables you to pick someone's prometheus and gather
info: samples scrapes, scrape duration, ... using *variables*.


Your dashboard should contain at least a singlestat panel, a variable and a graph
panel.

{{% info  %}}
Grafana also support tables and heapmaps for Prometheus
{{% /info %}}
