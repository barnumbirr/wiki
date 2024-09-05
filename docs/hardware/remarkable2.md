---
title: 'Remarkable 2'
date: 2021-08-27
tags: [ 'remarkable', 'paper', 'tablet', 'e-ink' ]
---

The Remarkable 2 or why you won't ever need paper notebooks again.

## Specifications

### Linux kernel

Running [`Software release 3.14`](https://support.remarkable.com/s/article/Software-release-3-14).


```bash
$ uname -a
Linux reMarkable 5.4.70-v1.3.4-rm11x #1 SMP PREEMPT Wed Mar 15 06:06:44 UTC 2023 armv7l GNU/Linux
```

??? "Older versions"

    Running [`Software release 3.9`](https://support.remarkable.com/s/article/Software-release-3-9).

    ```bash
    $ uname -a
    Linux reMarkable 5.4.70-v1.3.4-rm11x #1 SMP PREEMPT Wed Mar 15 06:06:44 UTC 2023 armv7l GNU/Linux
    ```

    Running [`Software release 2.9/2.10/2.11`](https://support.remarkable.com/s/article/Release-notes-overview).

    ```bash
    $ uname -a
    Linux reMarkable 5.4.70 #1 SMP PREEMPT Thu Aug 5 23:25:37 UTC 2021 armv7l GNU/Linux
    ```

### CPU information

```bash
$ sed '/^$/d' < /proc/cpuinfo | grep -m 1 'model name' | cut -c14-
ARMv7 Processor rev 5 (v7l)
```

??? "Full output"

        $ cat /proc/cpuinfo
        processor       : 0
        model name      : ARMv7 Processor rev 5 (v7l)
        BogoMIPS        : 16.00
        Features        : half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva
        idivt vfpd32 lpae evtstrm
        CPU implementer : 0x41
        CPU architecture: 7
        CPU variant     : 0x0
        CPU part        : 0xc07
        CPU revision    : 5

        processor       : 1
        model name      : ARMv7 Processor rev 5 (v7l)
        BogoMIPS        : 16.00
        Features        : half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva
        idivt vfpd32 lpae evtstrm
        CPU implementer : 0x41
        CPU architecture: 7
        CPU variant     : 0x0
        CPU part        : 0xc07
        CPU revision    : 5

        Hardware        : Freescale i.MX7 Dual (Device Tree)
        Revision        : 0000
        Serial          : 2638952245970

#### NXP™ i.MX7 Dual

NXP Semiconductors and Freescale Semiconductor [announced a merger agreement in
March 2015](https://en.wikipedia.org/wiki/NXP_Semiconductors).

- [Product Page](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-7-processors/i-mx-7dual-processors-heterogeneous-processing-with-dual-arm-cortex-a7-and-cortex-m4-cores:i.MX7D)
- [Datasheet](https://www.nxp.com/docs/en/data-sheet/IMX7DCEC.pdf)

### Memory information

```bash
$ cat /proc/meminfo
MemTotal:         994900 kB
MemFree:          356876 kB
MemAvailable:     798216 kB
Buffers:            8900 kB
Cached:           460332 kB
SwapCached:            0 kB
Active:           175044 kB
Inactive:         404864 kB
Active(anon):     110920 kB
Inactive(anon):    21156 kB
Active(file):      64124 kB
Inactive(file):   383708 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:                 0 kB
Writeback:             0 kB
AnonPages:        110672 kB
Mapped:            64732 kB
Shmem:             21404 kB
KReclaimable:       8928 kB
Slab:              18656 kB
SReclaimable:       8928 kB
SUnreclaim:         9728 kB
KernelStack:         864 kB
PageTables:         1340 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:      497448 kB
Committed_AS:     353108 kB
VmallocTotal:    1032192 kB
VmallocUsed:         844 kB
VmallocChunk:          0 kB
Percpu:              168 kB
CmaTotal:         327680 kB
CmaFree:          294544 kB
```

## Configuration

### Change SSH password

The default IP and SSH password of the Remarkable tablet can be found in the
settings menu.  
Navigate to it as follows: `Menu > Settings > General > Help > About >
Copyrights and licenses > General information > GPLv3 Compliance`.

The password is stored in `/home/root/.config/remarkable/xochitl.conf`.

```bash
[General]
DeveloperPassword=remarkable
```

??? "Older versions"

    Running [`Software release 2.x`](https://support.remarkable.com/s/article/Release-notes-overview).

    The password is stored in `/etc/remarkable.conf`.

### Fix SSH lag

Execute the following command on the device:

```bash
$ iw wlan0 set power_save off
```

### Updating timezone

The Remarkable's timezone is set to `UTC`.  
Due to unknown reasons, the reMarkable interface does not expose a visible
clock to the user. [remarkable-hacks](#additional-functionality) does expose a
clock to the user, hence why we need to set the correct timezone.

```bash
$ timedatectl set-timezone "Europe/Paris"
```

To see what timezones are available look at the contents of
`/usr/share/zoneinfo`.

### Additional functionality

#### Software release 2.x

[remarkable-hacks](https://github.com/ddvk/remarkable-hacks) adds a [literal
metric ton of
additional features](https://github.com/ddvk/remarkable-hacks#quick-doc) to the
Remarkable. Follow the installation instructions in the project's README.

#### Software release 3.x

[rm-hacks](https://github.com/mb1986/rm-hacks) seems to be the spiritual
successor to remarkable-hacks on software 3.x+.
