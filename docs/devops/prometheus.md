---
title: 'Prometheus'
date: 2021-08-27
tags: [ 'prometheus', 'monitoring', 'observability' ]
---

## Installation and updates

### Update binary installation

```bash
$ prometheus --version
prometheus, version 2.28.1 (branch: HEAD, revision: b0944590a1c9a6b35dc5a696869f75f422b107a1)
  build user:       root@2915dd495090
  build date:       20210701-15:20:10
  go version:       go1.16.5
  platform:         linux/amd64
$ curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -i -
$ tar xvf prometheus-2.29.2.linux-amd64.tar.gz
$ mv prometheus-2.29.2.linux-amd64/{prometheus,promtool} /usr/local/bin/
$ prometheus --version
prometheus, version 2.29.2 (branch: HEAD, revision: 752c4f11ae86effa9a46f017f2feb66730c67ed8)
  build user:       root@61bcc9848ade
  build date:       20210827-09:44:22
  go version:       go1.16.7
  platform:         linux/amd64
$ service prometheus restart
```

## Configuration

### Prometheus data retention

While Prometheus' data retention is set to 15 days by default, it can easily be
configured one of two ways: by `size` or by `time`.  

Size-based retention is set using the `--storage.tsdb.retention.size` flag, 
specifying a maximum amount of disk space used by blocks. Time-based retention
is set using the `--storage.tsdb.retention.time` flag, specifying the time
range which Prometheus will keep available. This is a minimum, so it'll keep an
entire block if some of it is still within the retention window.

Simply edit Prometheus' service file `/etc/systemd/system/prometheus.service`

```bash hl_lines="10"
[...]
[Service]
Type=simple
User=prometheus
Group=prometheus
#ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --storage.tsdb.retention.size=200GB \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url= \
  --web.enable-admin-api
[...]
```

### Probing multiple DNS servers with Blackbox exporter

```bash
  - job_name: 'DNS'
    scrape_interval: 300s
    metrics_path: /probe
    relabel_configs:
    # You can set internal labels that are called __param_<name>.
    # Those set URL parameter with the key <name> for the scrape request.
    # https://prometheus.io/docs/guides/multi-target-exporter/
    # Populate domain label with domain portion of __address__
      - source_labels: [__address__]
        regex: (.*):.*$
        replacement: $1
        target_label: domain
      # Populate instance label with dns server IP portion of __address__
      - source_labels: [__address__]
        regex: .*:(.*)$
        replacement: $1
        target_label: instance
      # Populate module URL parameter with domain portion of __address__
      # This is a parameter passed to the blackbox exporter
      - source_labels: [domain]
        target_label: __param_module
      # Populate target URL parameter with dns server IP
      - source_labels: [instance]
        target_label: __param_target
      # Populate __address__ with the address of the blackbox exporter to hit
      - target_label: __address__
        # Real address of blackbox exporter (Assumed to be running on port 9115 on localhost)
        replacement: localhost:9115
    static_configs:
    - targets:
      - example.com:9.9.9.9
      - example.com:1.1.1.1
      - example.com:127.0.0.1
```

<p style="font-size: 10px" align="right">
    Source: <a href="https://github.com/prometheus/blackbox_exporter/issues/51#issuecomment-385169368">prometheus/blackbox_exporter</a>
</p>

## Maintenance

### Delete timeseries metrics

Time series in Prometheus can be deleted over administrative HTTP API only
(disabled by default).

To enabled it, pass `--web.enable-admin-api` flag to Prometheus through start-up
script or docker-compose file, depending on installation method.

Use the following syntax to delete all time series metrics that match some
label:

```bash
$ curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={foo="bar"}'
```

To delete time series metrics that match some job or instance, run:

```bash
$ curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={job="node_exporter"}'
$ curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={instance="10.0.1.100:9100"}'
```

To delete all data from Prometheus, run:

```bash
$ curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={__name__=~".+"}'
```

Note that the above API calls don’t delete data immediately.

The actual data still exists on disk and will be cleaned up in future
compaction.

To determine when to remove old data, use
`--storage.tsdb.retention` option e.g. `--storage.tsdb.retention='365d'`.

To completely remove the data deleted by delete_series send clean_tombstones
API call:

```bash
$ curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/clean_tombstones'
```

<p style="font-size: 10px" align="right">
    <a href="https://www.shellhacks.com/prometheus-delete-time-series-metrics"/>Source</a>
</p>
