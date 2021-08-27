---
title: 'Icinga'
date: 2021-08-27
tags: [ 'icinga', 'monitoring', 'observability' ]
---

## Directories and file locations

### Plugins

```bash
$ /usr/lib/nagios/plugins
```

### Command definitions

```bash
$ /usr/share/icinga2/include/command-plugins.conf
```

## Show commands executed by Icinga

```bash
$ icinga2 feature enable debuglog
$ service icinga2 restart
$ tail -f /var/log/icinga2/debug.log
```

## List hosts

```
$ icinga2 object list --type Host
```
