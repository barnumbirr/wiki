---
title: 'Software RAID'
date: 2021-08-24
tags: [ 'mdadm', 'RAID', 'array', 'disk' ]
---

# mdadm: Linux software RAID

## Installation

```bash
apt-get install mdadm
```

## Check software RAID array status

Use the following command:

```bash
cat /proc/mdstat

or

mdadm -D /dev/mdX
```

Output in case of non-existant RAID device:

```bash
Personalities :
unused devices: <none>
```

Output for RAID6 device:

```bash
TODO
```

A progress bar is displayed under partitions when a RAID resync process is currently running:

```bash
Personalities : [raid6] [raid5] [raid4] [linear] [multipath] [raid0] [raid1] [raid10]
md0 : active raid6 sde[4] sdb[1] sdd[3] sda[0] sdc[2]
      35156259840 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]
      [=======>.............]  resync = 35.0% (4101838624/11718753280) finish=1179.1min speed=107662K/sec

unused devices: <none>
```

## Speed up software RAID resync

### Increase speed limits

Check current speed:

```bash
sysctl dev.raid.speed_limit_min
sysctl dev.raid.speed_limit_max
```
These values are set in Kibibytes per second (KiB/s).

Increase target rebuild speed:

```bash
echo 50000 > /proc/sys/dev/raid/speed_limit_min
echo 200000 > /proc/sys/dev/raid/speed_limit_max

or

sysctl -w dev.raid.speed_limit_min=50000
sysctl -w dev.raid.speed_limit_max=200000
```

Permanently override default values in `/etc/sysctl.conf`:

```bash
dev.raid.speed_limit_min = 50000
dev.raid.speed_limit_max = 200000
```

### Disable NCQ on all disks

```bash
for i in sd[abcde]
do
    echo 1 > /sys/block/$i/device/queue_depth
done
```

### Set read-ahead option

Get the current read-ahead value:

```bash
blockdev --getra /dev/mdX
```

Set read-ahead (in 512-byte sectors) per RAID device.

```bash
blockdev --setra 65536 /dev/mdX
```

### Increase stripe cache size

Records the size (in pages per device) of the stripe cache which is used for
synchronising all write operations to the array and all read operations if the
array is degraded. By default, the size of the stripe cache is 256 pages and
Linux uses 4096B pages. If you use 256 pages for the stripe cache and you have
10 disks, the cache would use 10*256*4096=10MiB of RAM.
*Only available on RAID5 and RAID6 and boost sync performance by 3-6 times.*

```bash
echo 4096 > /sys/block/mdX/md/stripe_cache_size
```
