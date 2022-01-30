---
title: 'Entware'
date: 2021-10-08
tags: [ 'Entware', Optware', 'opkg', 'ipkg', 'package' ]
---

## Usage

### Compile package from source

#### Install OpenWrt SDK build system dependencies

```bash
$ sudo apt-get update
$ sudo apt-get install build-essential ccache ecj fastjar file g++ gawk gettext\
git java-propose-classpath libelf-dev libncurses5-dev libncursesw5-dev\
libssl-dev python python2.7-dev python3 unzip wget python3-distutils\
python3-setuptools python3-dev rsync subversion swig time xsltproc zlib1g-dev
```

#### Build OpenWrt toolchain

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

The build output will be available under `bin/targets/<arch>`.

### Enabling debugging symbols when compililing package from source

`Entware (OpenWrt)` does not support debugging on device and therefor strips
all symbols during the installation phase of package creation. The only way to
debug programs on device is to use unstripped binaries on the build host and
`gdbserver` on device.

[Follow the instructions above](#compile-package-from-source) but stop before
compiling the package.

#### Enable compiling tools

In the directory the Entware repository was cloned to, run `menuconfig`

```bash
$ make menuconfig
```

Enable the `GNU debugger gdb` and `GNU debugger gdbserver` and as follows

```
Advanced configuration options (for developers) > Toolchain Options > Build gdb
Development > gdbserver
```

#### Add debugging to package

Add CFLAGS to the package Makefile and recompile.

```
TARGET_CFLAGS += -ggdb3
```

Alternatively recompile the package with CONFIG_DEBUG set.

```bash
$ make package/urbackup-server/compile -j$(nproc) V=s CONFIG_DEBUG=y
```

Or you can enable debug info in menuconfig

```
Global build settings > Compile packages with debugging info
```

#### Using the GNU debugger

Start `gdbserver` on the target device.

```bash
$ gdbserver :9000 urbackupsrv run -u admin -v debug
```

In the compiling tree, use the Entware (OpenWrt) `remote-gdb` script.

```bash
$ ./scripts/remote-gdb 10.0.1.100:9000 ./build_dir/target-x86_64_glibc-2.27/urbackup-server-2.4.13/urbackupsrv
```

You'll be dropped into a gdb shell and will be able to set breakpoints,
start program, backtraces, etc...

```
(gdb) b source-file.c:123
(gdb) c
(gdb) bt
```

If you want to restart the program, you'll need to set the remote path and
arguments.

```
(gdb) set remote exec-file /opt/bin/urbackupsrv
(gdb) set args run -u admin -v debug
(gdb) run
(gdb) bt
```

<p style="font-size: 12px" align="right">
    Source: <a href="https://openwrt.org/docs/guide-developer/gdb">OpenWrt Documentation</a>
</p>
