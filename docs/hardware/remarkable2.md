---
title: 'Remarkable 2'
date: 2021-08-27
tags: [ 'remarkable', 'paper', 'tablet', 'e-ink' ]
---

The Remarkable 2 or why you won't ever need paper notebooks again.

## Specifications

### Linux kernel

Running [`Software release 2.13`](https://support.remarkable.com/hc/en-us/sections/115001534689-Software-releases).

```bash
$ uname -a
Linux reMarkable 5.4.70-v1.1.5-rm11x #1 SMP PREEMPT Fri Nov 12 14:59:18 UTC 2021 armv7l GNU/Linux
```

??? "Older versions"

    Running [`Software release 2.9/2.10/2.11`](https://support.remarkable.com/hc/en-us/sections/115001534689-Software-releases).

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
MemTotal:        1027644 kB
MemFree:          468364 kB
MemAvailable:     851632 kB
Buffers:           12468 kB
Cached:           379288 kB
SwapCached:            0 kB
Active:           182876 kB
Inactive:         315920 kB
Active(anon):     107316 kB
Inactive(anon):     3540 kB
Active(file):      75560 kB
Inactive(file):   312380 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:                 0 kB
Writeback:             0 kB
AnonPages:        107036 kB
Mapped:            53052 kB
Shmem:              3820 kB
KReclaimable:      11580 kB
Slab:              20784 kB
SReclaimable:      11580 kB
SUnreclaim:         9204 kB
KernelStack:         792 kB
PageTables:         1372 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:      513820 kB
Committed_AS:     307304 kB
VmallocTotal:    1032192 kB
VmallocUsed:         968 kB
VmallocChunk:          0 kB
Percpu:              168 kB
CmaTotal:         327680 kB
CmaFree:          294640 kB
```

## Configuration

### Change SSH password

The default IP and SSH password of the Remarkable tablet can be found in the
settings menu.  
Navigate to it as follows: `Menu > Settings > Help > Copyrights and licenses > 
General information > GPLv3 Compliance`.

The password is stored in `/etc/remarkable.conf`.

```bash
[General]
DeveloperPassword=password
```

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

[remarkable-hacks](https://github.com/ddvk/remarkable-hacks) adds a [literal
metric ton of
additional features](https://github.com/ddvk/remarkable-hacks#quick-doc) to the
Remarkable. Follow the installation instructions in the project's README.
