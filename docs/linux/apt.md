---
title: 'apt'
date: 2021-08-25
tags: [ 'APT', package', 'Debian', 'Ubuntu' ]
---

# APT: Advanced Package Tool

## Retrieve package information

```bash
$ apt-cache show alacritty
Package: alacritty
Status: install ok installed
Priority: optional
Section: x11
Installed-Size: 5933
Maintainer: Martin Simon <email.obfuscated@example.com>
Architecture: amd64
Version: 0.9.0-1
Provides: x-terminal-emulator
Depends: libc6 (>= 2.29), libfontconfig1 (>= 2.12.6), libfreetype6 (>= 2.8), libgcc-s1 (>= 4.2), libxcb1 (>= 1.6)
Recommends: ncurses-term
Description: GPU-accelerated terminal emulator
 Alacritty is the fastest terminal emulator in existence. Using the GPU for
 rendering enables optimizations that simply aren't possible without it.
Description-md5: bf8f7998a09797b3ab5c3dbd67703aa2
Homepage: https://github.com/alacritty/alacritty
```

## Retrieve package dependencies

```bash
$ apt-cache showpkg alacritty
Package: alacritty
Versions:
0.9.0-1 (/var/lib/dpkg/status)
 Description Language:
                 File: /var/lib/dpkg/status
                  MD5: bf8f7998a09797b3ab5c3dbd67703aa2


Reverse Depends:
  ncurses-term,alacritty 0.3.4~
Dependencies:
0.9.0-1 - libc6 (2 2.29) libfontconfig1 (2 2.12.6) libfreetype6 (2 2.8) libgcc-s1 (2 4.2) libxcb1 (2 1.6) ncurses-term (0 (null))
Provides:
0.9.0-1 - x-terminal-emulator (= )
Reverse Provides:
```

## Retrieve package changelog

```bash
$ apt changelog alacritty
alacritty (0.9.0-1) unstable; urgency=medium

  * New upstream release
  * Remove manual terminfo file installation
  * Added `ncurses-term` to recommended package list

 -- Martin Simon <email.obfuscated@example.com>  Tue, 03 Aug 2021 00:00:00 +0000

alacritty (0.9.0-rc5-1) unstable; urgency=medium

  * New upstream release

 -- Martin Simon <email.obfuscated@example.com>  Mon, 02 Aug 2021 00:00:00 +0000

alacritty (0.9.0-rc4-1) unstable; urgency=medium

  * New upstream release

 -- Martin Simon <email.obfuscated@example.com>  Mon, 02 Aug 2021 00:00:00 +0000
[...]
```

## Replicate packages list on different hosts

On the source machine run:

```bash
$ apt-get update && apt-get dist-upgrade
$ cat /etc/apt/sources.list /etc/apt/sources.list.d/* > sources.list
$ dpkg --get-selections > selections.list
$ apt-mark auto > auto.list
```

On the target machine run:

```bash
$ cp sources.list /etc/apt/
$ apt-get update
$ dpkg --set-selections < selections.list
$ apt-get dselect-upgrade
$ xargs apt-mark auto < auto.list
```

The package list on your target machine should now match the one on the source
machine.
