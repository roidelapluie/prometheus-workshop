# Prometheus

Prometheus is an open source monitoring system designed around metrics. It is
a large ecosystem, with plenty of different components.

!!! Note
    The [prometheus documentation](https://prometheus.io/docs/introduction/overview/) provides an overview of those components


## The Prometheus server

1. Download the [prometheus server](https://prometheus.io/download/) 
2.5.0-rc.2.
1. Extract it

    ```
    $ tar xvf Downloads/prometheus-2.5.0-rc.2.linux-amd64.tar.gz
    ```

1. List the files

    ```
    $ ls prometheus-2.5.0-rc.2.linux-amd64
    ```

1. Launch prometheus

    ```
    $ cd prometheus-2.5.0-rc.2.linux-amd64
    ```

    ```
    $ ./prometheus
    ```

1. Open your browser at [http://127.0.0.1:9090](http://127.0.0.1:9090)

1. Look at the TSDB data

!!! note
    [prometheus/tsdb](https://github.com/prometheus/tsdb) is the database
    backend of prometheus. It is maintained by
    the prometheus team.


## The web ui

There is a lot of information that can be found in the prometheus server web ui.

Try to find:

- The version of prometheus
- The duration of data retention
- The "targets" that are scraped by default
- The "scrape" interval

## promtool

promtool is a command line tool provided with Prometheus.

With promtool you can:

- Query Prometheus

  ```
  ./promtool query instant http://127.0.0.1:9090 up
  ```

!!! info
    the *up* metric is added by prometheus on each scrape. It is 1 if the scrape
    has succeeded, 0 otherwise.

- Validate Prometheus configuration

   ```
   ./promtool check config prometheus.yml
   ```

## Adding targets

!!! note
    At this point, make sure you understand the basis of YAML

*exercise*

- Open prometheus.yml
- Add each one's prometheus server as targets to your prometheus server.
- Look the status (using up or the target page)


What is a job? What is an instance?

!!! tip
    You do not need to reload prometheus: you can just send a `SIGHUP` signal to
    reload the configuration:

    ```
    killall -HUP prometheus
    ```

## Admin commands

1. Enable admin commmands

    ```
    $ ./prometheus --web.enable-admin-api
    ```

1. Take a snapshot

    ```
    $ curl -XPOST http://localhost:9090/api/v1/admin/tsdb/snapshot
    ```

    Look in the data directory

1. Delete a timeserie

    ```
    $ curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]=process_start_time_seconds{job="prometheus"}'
    ```

## Federation

Now, let's move to file_sd.

Create a file:

```
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

```
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

!!! tip
    The name of a metric is a label too! It is the `__name__` label.

??? success "Solution"

    ```
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



## Last exercise

Prometheus fetches Metrics as HTML.

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
