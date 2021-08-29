---
title: 'Helios64'
date: 2021-08-27
tags: [ 'helios64', 'kobol.io','Network Attached Storage', 'NAS' ]
---

The Helios64 is a powerful ARM board specially designed for Network Attached
Storage (NAS). It harnesses its processing capabilities from the Rockchip
RK3399 SoC.

## Increase serial console verbosity

### Boot from microSD card

Add the following lines to `/boot/armbianEnv.txt`

```bash
verbosity=7
console=serial
extraargs=earlyprintk ignore_loglevel
```

### Boot from eMMC

Boot from a fresh image on a microSD card, mount the eMMC then modify
`/boot/armbianEnv.txt`.

## Log serial console output to file

```bash
$ screen -S helios64-serial
$ picocom -b 1500000 /dev/ttyUSB0 -g <filename>.txt
```

## Start fans during early boot

Allow the fans to spin at a constant speed from the earliest stage of initrd
until fancontrol is started by the system. If the boot process encounters an
error, the fans may never start and your device will overheat.

Include the pwm-fan module to initramfs like so:

```bash
$ echo pwm-fan | sudo tee -a /etc/initramfs-tools/modules
```

Create `/etc/initramfs-tools/scripts/init-top/fan`:

```bash
#!/bin/sh

PREREQ=""
prereqs() {
        echo "$PREREQ"
}

case $1 in
prereqs)
        prereqs
        exit 0
        ;;
esac

. /scripts/functions

modprobe pwm-fan

for pwm in /sys/devices/platform/p*-fan/hwmon/hwmon*/pwm1; do
        echo 150 >$pwm
done

exit 0
```

The `$pwm` value can be set anywhere between 1 and 255 and specifies how fast
the fans should spin.

Include the fan script in initramfs and update the initramfs image:

```bash
$ sudo chmod +x /etc/initramfs-tools/scripts/init-top/fan
$ sudo update-initramfs -u -k all
$ sudo reboot
```

## Update bootloader

```bash
$ nand-sata-install
# select option 5 "Install/Update the bootloader on SD/eMMC"
```

## Disable NCQ

Edit `/boot/armbianEnv.txt` and add the following

```bash
extraargs=libata.force=noncq
```

## Audit ArmbianEnv.txt

For some unknown reason `/boot/armbianEnv.txt` keeps getting overwritten with
logrotate configuration "gibberish". To monitor which process is modifying the
file, setup `autitd` as follows:

```bash
$ sudo apt-get install auditd
$ sudo auditctl -w /boot/armbianEnv.txt -p wa
$ sudo tail -F /var/log/audit/audit.log
```

The output should show actions performed on the file and the process ID
performing those actions.

## Fan management

The standard fans shipped with the Helios64 were rather noisy and therefor
replaced by two Noctua NF-A8 PWM fans. A custom fancontrol configuration
was also implemented:

```bash
$ cat /etc/fancontrol
# Helios64 PWM Fan Control Configuration
# Temp source : /dev/thermal-cpu
INTERVAL=10
FCTEMPS=/dev/fan-p6/pwm1=/dev/thermal-cpu/temp1_input /dev/fan-p7/pwm1=/dev/thermal-cpu/temp1_input
MINTEMP=/dev/fan-p6/pwm1=40 /dev/fan-p7/pwm1=40
MAXTEMP=/dev/fan-p6/pwm1=75 /dev/fan-p7/pwm1=75
MINSTART=/dev/fan-p6/pwm1=125 /dev/fan-p7/pwm1=125
MINSTOP=/dev/fan-p6/pwm1=100 /dev/fan-p7/pwm1=100
MINPWM=125
```
## systemd-networkd-wait-online

By default, `systemd-networkd-wait-online.service` waits for all links it is
aware of and which are managed by *systemd-networkd* to be fully configured or
failed, and for at least one link to be online.

With the Helios64 having dual Ethernet ports (but only one cable plugged in),
starting `systemd-networkd-wait-online.service` will fail after the default
timeout of 2 minutes. This may cause an unwanted delay in the startup process.
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

<p style="font-size: 10px" align="right">
    Source: <a href="https://wiki.archlinux.org/title/Systemd-networkd#systemd-networkd-wait-online">wiki.archlinux.org</a>
</p>
