---
title: 'Prometheus'
date: 2021-08-27
tags: [ 'prometheus', 'monitoring','observability' ]
---

## Update Prometheus

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
