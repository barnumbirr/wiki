---
title: 'Xiaomi Mi WiFi R3G'
date: 2021-09-26
tags: [ 'Xiaomi', 'router']
---

## Specifications

### Linux kernel

Running `Padavan 3.4.3.9L-101_e662678`.

```bash
$ uname -a
Linux ap1 3.4.113 #1 SMP Sat May 15 10:43:11 +07 2021 mips GNU/Linux
```

### CPU information

```bash
$ sed '/^$/d' < /proc/cpuinfo | grep -m 1 'system type' | cut -c16-
MediaTek MT7621 SoC
```

??? "Full output"

        $ cat /proc/cpuinfo
        system type             : MediaTek MT7621 SoC
        processor               : 0
        cpu model               : MIPS 1004Kc V2.15
        BogoMIPS                : 583.68
        wait instruction        : yes
        microsecond timers      : yes
        tlb_entries             : 32
        extra interrupt vector  : yes
        hardware watchpoint     : yes, count: 4, address/irw mask: [0x0000, 0x0000, 0x0000, 0x0000]
        ASEs implemented        : mips16 dsp mt
        shadow register sets    : 1
        kscratch registers      : 0
        core                    : 0
        VPE                     : 0
        VCED exceptions         : not available
        VCEI exceptions         : not available

        processor               : 1
        cpu model               : MIPS 1004Kc V2.15
        BogoMIPS                : 583.68
        wait instruction        : yes
        microsecond timers      : yes
        tlb_entries             : 32
        extra interrupt vector  : yes
        hardware watchpoint     : yes, count: 4, address/irw mask: [0x0000, 0x0000, 0x0000, 0x0000]
        ASEs implemented        : mips16 dsp mt
        shadow register sets    : 1
        kscratch registers      : 0
        core                    : 0
        VPE                     : 1
        VCED exceptions         : not available
        VCEI exceptions         : not available

        processor               : 2
        cpu model               : MIPS 1004Kc V2.15
        BogoMIPS                : 583.68
        wait instruction        : yes
        microsecond timers      : yes
        tlb_entries             : 32
        extra interrupt vector  : yes
        hardware watchpoint     : yes, count: 4, address/irw mask: [0x0000, 0x0ff8, 0x0000, 0x0000]
        ASEs implemented        : mips16 dsp mt
        shadow register sets    : 1
        kscratch registers      : 0
        core                    : 1
        VPE                     : 0
        VCED exceptions         : not available
        VCEI exceptions         : not available

        processor               : 3
        cpu model               : MIPS 1004Kc V2.15
        BogoMIPS                : 583.68
        wait instruction        : yes
        microsecond timers      : yes
        tlb_entries             : 32
        extra interrupt vector  : yes
        hardware watchpoint     : yes, count: 4, address/irw mask: [0x0000, 0x0000, 0x0000, 0x0000]
        ASEs implemented        : mips16 dsp mt
        shadow register sets    : 1
        kscratch registers      : 0
        core                    : 1
        VPE                     : 1
        VCED exceptions         : not available
        VCEI exceptions         : not available

#### MediaTek MT7621 SoC

- [Product Page](https://datasheetspdf.com/pdf-file/1144680/Ralink/MT7621/1)

### Memory information

```bash
$ cat /proc/meminfo
MemTotal:         256088 kB
MemFree:          223560 kB
Buffers:            3276 kB
Cached:             8420 kB
SwapCached:            0 kB
Active:             5796 kB
Inactive:           7860 kB
Active(anon):       2120 kB
Inactive(anon):      156 kB
Active(file):       3676 kB
Inactive(file):     7704 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:                 0 kB
Writeback:             0 kB
AnonPages:          1992 kB
Mapped:             2424 kB
Shmem:               304 kB
Slab:               9804 kB
SReclaimable:       1248 kB
SUnreclaim:         8556 kB
KernelStack:         400 kB
PageTables:          224 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:      128044 kB
Committed_AS:       5472 kB
VmallocTotal:    1048372 kB
VmallocUsed:        5084 kB
VmallocChunk:    1041136 kB
```
