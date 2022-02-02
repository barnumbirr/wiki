---
title: 'GRUB'
date: 2021-10-04
tags: [ 'grub', 'bootloader' ]
---

# GRUB - GRand Unified Bootloader

## Configuration

!!! hint
    If you've changed grub configuration file `/etc/default/grub`, make sure to
    run `update-grub` to update its configuration.

### Configuring non-default/older kernel to boot

#### Save selected kernel in console menu

Edit `/etc/default/grub` and add the following lines:

```bash
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
```

Reboot the device and select an entry in the GRUB menu/submenu. The selection
will be saved as the default value.
Manually selecting a different entry updates the saved value to become the new
default.

#### Set kernel version in config file

Set e.g:

```bash
GRUB_DEFAULT="1>2"
```

!!! note
    GRUB menu entries numbering starts with 0. Therefore the `1` above points
    to the `Advanced` submenu and `2` points to the third entry of the submenu.

### Themes

Prefered GRUB themes are [`shvchk/poly-dark`](https://github.com/shvchk/poly-dark),
[`shvchk/poly-light`](https://github.com/shvchk/poly-light) or
[`vinceliuice/grub2-themes`](https://github.com/vinceliuice/grub2-themes)
