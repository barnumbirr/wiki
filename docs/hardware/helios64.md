---
title: 'Helios64 Open Source NAS'
date: 2021-08-25
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
