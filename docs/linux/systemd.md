---
title: 'systemd'
date: 2021-08-27
tags: [ 'systemd', 'system manager', 'service manager', 'init system', 'process' ]
---

## Configuration

### systemd-networkd-wait-online

By default, `systemd-networkd-wait-online.service` waits for all links it is
aware of and which are managed by *systemd-networkd* to be fully configured or
failed, and for at least one link to be online.

If your system has multiple network interfaces, but some are not expected to be
connected all the time (e.g. if you have a dual-port Ethernet card, but only one
cable plugged in), starting `systemd-networkd-wait-online.service` will fail
after the default timeout of 2 minutes. This may cause an unwanted delay in the
startup process.
To change the behaviour to wait for any interface rather than all interfaces to
become online, edit the service and add the `--any` parameter to the `ExecStart`
line:

```bash hl_lines="22"
$ cat /etc/systemd/system/network-online.target.wants/systemd-networkd-wait-online.service
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Wait for Network to be Configured
Documentation=man:systemd-networkd-wait-online.service(8)
DefaultDependencies=no
Conflicts=shutdown.target
Requires=systemd-networkd.service
After=systemd-networkd.service
Before=network-online.target shutdown.target

[Service]
Type=oneshot
ExecStart=/lib/systemd/systemd-networkd-wait-online --any
RemainAfterExit=yes

[Install]
WantedBy=network-online.target
```

!!! note
    This issue occurs on the [Helios64](../hardware/helios64.md).

<p style="font-size: 10px" align="right">
    Source: <a href="https://wiki.archlinux.org/title/Systemd-networkd#systemd-networkd-wait-online">wiki.archlinux.org</a>
</p>
