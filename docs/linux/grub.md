---
title: 'GRUB - GRand Unified Bootloader'
date: 2021-10-04
tags: [ 'grub', bootloader' ]
---

## Configuration

!!! hint
    If you've changed grub configuration file `/etc/default/grub`, make sure to
    run `update-grub` to update its configuration.

### Set default kernel

#### Save selected entry

Edit `/etc/default/grub` and add the following lines:

```bash
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
```
Don't forget to comment out the following:

```bash
# GRUB_DEFAULT=0
```

Reboot the device and select an entry in the GRUB menu/submenu. The selection
will be saved as the default value.
Manually selecting a different entry updates the saved value to become the new
default.
