---
title: 'fwupd'
date: 2021-08-25
tags: [ 'firmware', 'bios', 'LVFS' ]
---

fwupd is a simple daemon allowing you to update some devices' firmware,
including UEFI for several machines.

## Installation

```bash
$ apt install fwupd
```

## Usage

Display all detected devices, the output below is from a Thinkpad A485:

```bash
$ fwupdmgr get-devices
20MUCTO1WW
│
├─INTEL SSDPEKKF256G8L:
│     Device ID:          3743975ad7f64f8d6575a9ae49fb3a8856fe186f
│     Summary:            NVM Express Solid State Drive
│     Current version:    L08P
│     Vendor:             Intel Corporation (NVME:0x8086)
│     GUIDs:              f91a5c60-8696-539e-9d0a-57d194f74ac4
│                         396fb4fd-7656-564f-88b5-381e83ca30c0
│                         79517f86-8df8-5d6e-a18b-33f0b36a78e9
│                         68db11e5-b0cf-5bc9-a94e-17e28496e505
│                         cdc79848-b022-5a05-a77a-862f21df0c29
│     Device Flags:       • Internal device
│                         • Updatable
│                         • System requires external power source
│                         • Needs a reboot after installation
│                         • Device is usable for the duration of the update
│
├─System Firmware:
│ │   Device ID:          ed52bde4c2fa3be2e57206a51ecb00d0219d6bcb
│ │   Current version:    0.1.45
│ │   Minimum Version:    0.1.45
│ │   Vendor:             LENOVO (DMI:LENOVO)
│ │   GUIDs:              ff8bc5c6-2169-42b7-907b-c726a9c2dab2
│ │                       230c8b18-8d9b-53ec-838b-6cfc0383493a
│ │                       8cf29274-0132-594f-a561-02696eb4eda4
│ │   Device Flags:       • Internal device
│ │                       • Updatable
│ │                       • System requires external power source
│ │                       • Supported on remote server
│ │                       • Needs a reboot after installation
│ │                       • Cryptographic hash verification is available
│ │                       • Device is usable for the duration of the update
│ │
│ └─UEFI dbx:
│       Device ID:        362301da643102b9f38477387e2193e57abaa590
│       Summary:          UEFI Revocation Database
│       Current version:  77
│       Minimum Version:  77
│       Vendor:           UEFI:Linux Foundation
│       Install Duration: 1 second
│       GUIDs:            0477b68d-b0db-5cb9-bfe2-ea54879b925a
│                         60f9de84-e7a3-5f5c-b0e8-cc39d95f865b
│                         c6682ade-b5ec-57c4-b687-676351208742
│                         f8ba2887-9411-5c36-9cee-88995bb39731
│       Device Flags:     • Internal device
│                         • Updatable
│                         • Needs a reboot after installation
│
├─UEFI Device Firmware:
│     Device ID:          d40c74fc1cd9aaa8423ea15ee7dcdb5b07cf808b
│     Current version:    4294967152
│     Minimum Version:    4294967152
│     Vendor:             DMI:LENOVO
│     GUIDs:              7f2df857-c3eb-44aa-b906-39cd03170770
│                         0de7e49d-1183-5a9f-abcd-77817b577ed5
│     Device Flags:       • Internal device
│                         • Updatable
│                         • System requires external power source
│                         • Needs a reboot after installation
│                         • Device is usable for the duration of the update
│
├─UEFI Device Firmware:
│     Device ID:          4354ccbb577b307820024ceb0c278e0a276a973d
│     Current version:    16909369
│     Minimum Version:    1
│     Vendor:             DMI:LENOVO
│     GUIDs:              1434e005-f4d0-438a-b1a4-b649c051a796
│                         ec5a37ae-78fd-51d4-9d1a-ef435f91d3a0
│     Device Flags:       • Internal device
│                         • Updatable
│                         • System requires external power source
│                         • Needs a reboot after installation
│                         • Device is usable for the duration of the update
│
├─UEFI Device Firmware:
│     Device ID:          4de501752c7423618c1a1376cd4b14f399827ede
│     Current version:    17040444
│     Minimum Version:    1
│     Vendor:             DMI:LENOVO
│     GUIDs:              0655db59-d680-49d3-863d-83d5bac59568
│                         c5000566-551f-5380-9a27-6a8e242999cf
│     Device Flags:       • Internal device
│                         • Updatable
│                         • System requires external power source
│                         • Needs a reboot after installation
│                         • Device is usable for the duration of the update
│
├─UEFI Device Firmware:
│     Device ID:          5d5bea04759f7663b03266fad519d98e5d51a140
│     Current version:    4784132
│     Minimum Version:    4784132
│     Vendor:             DMI:LENOVO
│     GUIDs:              4fcecda9-f412-4623-b603-fb6a9495df5b
│                         a2bd78fe-2b8a-50d6-bc3d-c7b28834e956
│     Device Flags:       • Internal device
│                         • Updatable
│                         • System requires external power source
│                         • Needs a reboot after installation
│                         • Device is usable for the duration of the update
│
└─UEFI Device Firmware:
      Device ID:          0309fe7d84b0b8e838ad1b2f22bd153906121cfc
      Current version:    65569
      Minimum Version:    1
      Vendor:             DMI:LENOVO
      GUIDs:              4bea12df-56e3-4cdb-97dd-f133768c9051
                          31bfb296-9f8b-5519-94ad-04dadf19d202
      Device Flags:       • Internal device
                          • Updatable
                          • System requires external power source
                          • Needs a reboot after installation
                          • Device is usable for the duration of the update
```

Download the latest metadata from the
[Linux Vendor firmware Service (LVFS)](https://fwupd.org/):

```bash
$ fwupdmgr refresh
Updating lvfs
Downloading…             [***************************************]
Successfully downloaded new metadata: 1 local device supported
```

List updates available for devices on system:

```bash
$ fwupdmgr get-updates
Devices with no available firmware updates:
 • INTEL SSDPEKKF256G8L
 • UEFI Device Firmware
 • UEFI Device Firmware
 • UEFI Device Firmware
 • UEFI Device Firmware
 • UEFI Device Firmware
 • UEFI dbx
Devices with the latest available firmware version:
 • System Firmware
No updates available for remaining devices
```

Install updates:

```bash
$ fwupdmgr update
```

!!! note
    - Some device updates may require the root user
    - Updates that can be applied live will be done immediately.
    - Updates that run at bootup will be staged for the next reboot.
