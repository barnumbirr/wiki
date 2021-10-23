---
title: 'Entware'
date: 2021-10-08
tags: [ 'Entware', Optware', 'opkg', 'ipkg', 'package' ]
---

## Usage

### Compile package from source

#### Install OpenWRT SDK build system dependencies

```bash
$ sudo apt-get update
$ sudo apt-get install build-essential ccache ecj fastjar file g++ gawk gettext\
git java-propose-classpath libelf-dev libncurses5-dev libncursesw5-dev\
libssl-dev python python2.7-dev python3 unzip wget python3-distutils\
python3-setuptools python3-dev rsync subversion swig time xsltproc zlib1g-dev
```

#### Build OpenWRT toolchain

This is the cross-compilation toolchain for the
[QNAP TS-873A](../hardware/TS873A.md) NAS.

```bash
$ git clone https://github.com/Entware/Entware.git
$ cd Entware
$ make package/symlinks
$ cp configs/x64-3.2.config .config
$ make tools/install -j$(nproc)
$ make toolchain/install -j$(nproc)
$ make target/compile -j$(nproc)
```

#### Customize package Makefile

```bash
$ nano Entware/feeds/rtndev/urbackup-server/Makefile
TARGET_CFLAGS += -ggdb3
```

#### Compile package

```bash
$ cd Entware
$ make package/urbackup-server/compile -j$(nproc) V=s
```
