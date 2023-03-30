---
title: 'opkg'
date: 2021-09-11
tags: [ 'opkg', 'ipkg', 'package', 'Entware', 'Optware' ]
---

`opkg` is a lightweight package manager designed to add software to stock
firmware of embedded devices.

!!! note
    `opkg` is the package manager used on the
    [QNAP TS-873A](../hardware/TS873A.md).

## Installation

Open the QNAP App Center, add the Qnapclub repository and install the
[`Entware-std`](https://www.qnapclub.eu/en/qpkg/556) package. For more
information between Entware's `standard` and `alternative` installation,
[refer to the dedicated Entware wiki page](https://github.com/Entware/Entware/wiki/Alternative-install-vs-standard).

## Configuration

### Download packages via TLS

Install wget as `opkg` uses it to fetch packages from repositories:

```bash
$ opkg install wget ca-certificates
```

Update `opkg` repository URL:

```bash
$ sed -i 's/http:\/\//https:\/\//g' /opt/etc/opkg.conf
```

## Usage

### Fix broken packages after `opkg upgrade`

```bash
$ opkg update
$ opkg upgrade
$ minidlna
  minidlna: error while loading shared libraries: libpng12.so.0: cannot open shared object file: No such file or director
```

Unfortunately, opkg isn't sophisticated enough to react to the fact that the
minidlna dependencies have changed and that minidlna should be re-installed too.
Instead, you have to do that manually:


```bash
$ opkg --autoremove remove minidlna
$ opkg install minidlna
```

As a last resort, if the solution provided above doesn't work, try the following:


```bash
$ opkg list-installed | grep -v "libc " | sed 's/ - .*$//' | grep ^lib | grep -v libpthread | grep -v libgcc | xargs -n 5 opkg --force-reinstall install
```
