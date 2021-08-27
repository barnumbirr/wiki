---
title: 'Prometheus'
date: 2021-08-27
tags: [ 'prometheus', 'monitoring','observability' ]
---

## Configure Prometheus data retention

While Prometheus' data retention is set to 15 days by default, it can easily be
configured one of two ways: by `size` or by `time`.  

Size-based retention is set using the `--storage.tsdb.retention.size` flag, 
specifying a maximum amount of disk space used by blocks. Time-based retention
is set using the `--storage.tsdb.retention.time` flag, specifying the time
range which Prometheus will keep available. This is a minimum, so it'll keep an
entire block if some of it is still within the retention window.

Simply edit Prometheus' service file `/etc/systemd/system/prometheus.service`

``` bash hl_lines="10"
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
