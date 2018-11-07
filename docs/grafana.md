# Grafana

## Download and run grafana

1. Go to the [Grafana website](https://grafana.com/) and download the latest stable release
1. For this exercise we will use the Standalone Linux Binaries(64 Bit)
  ```
  $ wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.3.2.linux-amd64.tar.gz
  $ tar -zxvf grafana-5.3.2.linux-amd64.tar.gz
  $ cd grafana-5.3.2
  $ ./bin/grafana-server
  ```
1. Open Grafana in your browser


## Setup prometheus

Add your prometheus server as a datasource + import the prometheus dashboards

Monitor grafana in prometheus (add it as a target)

Look at the grafana dashboards


## Create a new dashboard

Create a new dashboard which enables you to pick someone's prometheus and gather
info: samples scrapes, scrape duration, ... using *variables*.


Your dashboard should contain at least a singlestat panel, a variable and a graph
panel.

!!! info
    Grafana also support tables and heapmaps for Prometheus
