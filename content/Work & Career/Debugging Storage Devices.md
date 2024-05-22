## Context

The machine runs a really simple web service that serves objects out of a directory. Feel free to check it out by running something like `curl http://localhost:8000`, but the objective is really just to approximate a production service that relies on some disk storage.

We're pretending we have real disks in our LVM pool by using loopback devices on top of files in the `/media` directory. We're just creating them with [`fallocate -l 1800M /media/diskN.img`](https://man7.org/linux/man-pages/man1/fallocate.1.html) as 1800MB images.

```
root@6e82dd90c22398:/# ls -lh /media/disk0.img
-rw-r--r--  1 root root 1887436800 Aug 23 17:05 disk0.img
```

We ran out of space so we created a few more "disks" and added them to the pool so now there are 3, but things still aren't working quite right. Then we asked ChatGPT for help and are pretty certain we've managed to break things even more. So we need your help getting things healthy again, and helping ensure we don't run out of disk space as more data is created.

The objective:

1. Fix up the **existing** `/data` volume so it's usable.
2. Set up the `vg0` VG with four 1800MB disks, using loopback images.
3. Expand the `data0` LV to 4GB.
4. Write a tiny bit of software to automatically expand the volume by 500M any time it gets close to full.

On the host is a script that will help you test things as you make progress. You can run `test-storage.sh` and it will run some simple tests to ensure things are working.
## Commands to replicate (copy+paste) 

```shell
pvchange --allocatable y /dev/loop0
mount -o rw,remount /data

pvresize --verbose /dev/loop1 --setphysicalvolumesize 1800M
pvresize --verbose /dev/loop2 --setphysicalvolumesize 1800M

pvmove --alloc anywhere /dev/loop1:0-12
vgreduce -A y vg0 /dev/loop1
pvcreate /dev/loop1
vgextend vg0 /dev/loop1
lvextend -L 4G /dev/mapper/vg0-data0 -A y --resizefs

pid=$(ps aux | grep server | awk 'NR==1{print $2}')
kill ${pid}
sleep 5

fallocate -l 1800M /media/disk3.img
losetup /dev/loop3 /media/disk3.img
pvcreate /dev/loop3
vgextend  vg0 /dev/loop3

## copypaste script & run storage test
```

## Fixing /data
what exactly is wrong with `/data`?

Looking at `test-storage.sh`, seems like `/data` is unwriteable.

The LV itself has the writeable attribute.

```shell
root@2874d33a027178:/data# lvs
  LV    VG  Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  data0 vg0 -wi-ao---- 2.80g
```


We can unmount `/data` and try running `fsck` on the LVM:
```shell
root@9080d7da72e587:/data# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs                97M     0   97M   0% /dev
/dev/vda               7.8G  7.2G  232M  97% /
shm                    109M     0  109M   0% /dev/shm
tmpfs                  109M     0  109M   0% /sys/fs/cgroup
/dev/mapper/vg0-data0  2.7G  628M  2.0G  25% /data

root@9080d7da72e587:/# umount -v data
umount: /data (/dev/mapper/vg0-data0) unmounted

root@9080d7da72e587:/# fsck -V /dev/vg0/data0
fsck from util-linux 2.34
[/usr/sbin/fsck.ext4 (1) -- /data] fsck.ext4 /dev/mapper/vg0-data0
e2fsck 1.45.5 (07-Jan-2020)
/dev/mapper/vg0-data0: clean, 160044/183632 files, 190762/734208 blocks

root@9080d7da72e587:/# mount /dev/mapper/vg0-data0 /data
root@9080d7da72e587:/# cd /data
root@9080d7da72e587:/data# mkdir test
root@9080d7da72e587:/data# ls
lost+found  objects  test
```

It works now, but it doesn't seem right that `fsck` didn't report anything out of the ordinary. Since the LVM has the writable attribute, let's  recreate the machine & check the mount options:

```shell
# the act of remounting fixed it, not fsck
root@5683e7ea34268e:/# cat /proc/mounts | grep data
/dev/mapper/vg0-data0 /data ext4 ro,relatime 0 0
```

Non-disruptive way to change mount to rw

```shell
root@e286573cd16186:/# mount -o rw,remount /data
```

## Setting PV's to their full size

I was going to allocate another disk as shown in the README, but it looks like we don't have enough space on the root partition to do so. Let's tackle it later. The `/dev/loop1` and `/dev/loop2` aren't showing as 1.75G, this is a low hanging fruit.

```shell
# physical volumes aren't the right size
root@e286573cd16186:/media# pvs
  PV         VG  Fmt  Attr PSize    PFree
  /dev/loop0 vg0 lvm2 u--     1.75g      0
  /dev/loop1 vg0 lvm2 a--   776.00m 724.00m
  /dev/loop2 vg0 lvm2 a--  1020.00m      0

root@e286573cd16186:/# pvresize --verbose /dev/loop1 --setphysicalvolumesize 1800M
  Archiving volume group "vg0" metadata (seqno 3).
  /dev/loop1: Size is already <1.76 GiB (3686400 sectors).
  Resizing volume "/dev/loop1" to 3686400 sectors.
  No change to size of physical volume /dev/loop1.
  Updating physical volume "/dev/loop1"
  Creating volume group backup "/etc/lvm/backup/vg0" (seqno 4).
  Physical volume "/dev/loop1" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized

root@e286573cd16186:/# pvresize --verbose /dev/loop2 --setphysicalvolumesize 1800M
  Archiving volume group "vg0" metadata (seqno 4).
  /dev/loop2: Size is already <1.76 GiB (3686400 sectors).
  Resizing volume "/dev/loop2" to 3686400 sectors.
  Resizing physical volume /dev/loop2 from 255 to 449 extents.
  Updating physical volume "/dev/loop2"
  Creating volume group backup "/etc/lvm/backup/vg0" (seqno 5).
  Physical volume "/dev/loop2" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized
```

Even after resizing, `/dev/loop1` still doesn't seem to be the right size.

## Recreating PV /dev/loop1

Looking at `pvdisplay`, we can see that 1.0GB is not usable in the PV size. That's a lot, when the others only use ~4MB. Let's recreate the loop1 pv so that it can use all the space it can.

```shell
# I lost the output, but also checked the pe_start of loop1 seeing that it was starting at a significantly
# later extent than the others!

root@e286573cd16186:~# pvdisplay -v -m
...pv loop0...
  --- Physical volume ---
  PV Name               /dev/loop1
  VG Name               vg0
  PV Size               <1.76 GiB / not usable 1.00 GiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              194
  Free PE               181
  Allocated PE          13
  PV UUID               oOXTtP-wVRT-j8uA-w5VN-3Lb7-jTUo-deB8gE

  --- Physical Segments ---
  Physical extent 0 to 12: # good thing it's only using 48 mib!
    Logical volume      /dev/vg0/data0
    Logical extents     704 to 716
  Physical extent 13 to 193:
    FREE
...pv loop1...

root@e286573cd16186:~# pvmove --alloc anywhere /dev/loop1:0-12
  /dev/loop1: Moved: 30.77%
  /dev/loop1: Moved: 100.00%

# verifying that everything is moved off
root@e286573cd16186:~# pvs
  PV         VG  Fmt  Attr PSize   PFree
  /dev/loop0 vg0 lvm2 u--    1.75g      0
  /dev/loop1 vg0 lvm2 a--  776.00m 776.00m
  /dev/loop2 vg0 lvm2 a--    1.75g 724.00m
root@e286573cd16186:~# vgreduce -A y vg0 /dev/loop1
root@e286573cd16186:~# pvcreate /dev/loop1
  Physical volume "/dev/loop1" successfully created.
root@e286573cd16186:~# vgextend vg0 /dev/loop1
  Volume group "vg0" successfully extended
root@e286573cd16186:~# lvextend -L 4G /dev/mapper/vg0-data0 -A y
  Rounding size to boundary between physical extents: 1.20 GiB.
  Size of logical volume vg0/data0 changed from 2.80 GiB (717 extents) to 4.00 GiB (1025 extents).
  Logical volume vg0/data0 successfully resized.
# forgot to resize actual volume at this point, ended up resizing later

# pv's correctly allocated & lv is right size
root@e286573cd16186:~# pvs
  PV         VG  Fmt  Attr PSize PFree
  /dev/loop0 vg0 lvm2 u--  1.75g     0
  /dev/loop1 vg0 lvm2 a--  1.75g <1.26g
  /dev/loop2 vg0 lvm2 a--  1.75g     0
root@e286573cd16186:~# lvs
  LV    VG  Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  data0 vg0 -wi-ao---- 4.00g
```

## Adding another disk

[Googling reasons why `df` might not match free size with actual size](https://askubuntu.com/questions/249387/df-h-used-space-avail-free-space-is-less-than-the-total-size-of-home), it suggested checking for any recently deleted files.  

```shell
root@e286573cd16186:/# lsof +L1
COMMAND PID USER   FD   TYPE DEVICE   SIZE/OFF NLINK NODE NAME
python3 381 root    3r   REG  254,0 3984588800     0 4545 /.tempfile (deleted)

# server is still holding onto ~3GB, let's investigate how to safely restart
root@e286573cd16186:/# ps aux
... other output ...
root       462  0.5  6.5  20476 14624 ?        S    23:36   0:00 python3 /usr/local/bin/server

root@e286573cd16186:/usr/local/bin# cat server
... contents ...
# figured out that it will politely exit, we can do regular kill signal

root@e286573cd16186:/usr/local/bin# cat run.sh
... contents ...
# this continuously keeps the server up, seems safe to kill
root@e286573cd16186:/usr/local/bin# kill 381
root@e286573cd16186:/usr/local/bin# ps aux # double check that it is indeed dead, or at least diff PID now
root@e286573cd16186:/usr/local/bin# df -h # all this beautiful space now
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs                97M     0   97M   0% /dev
/dev/vda               7.8G  3.5G  4.0G  47% /
shm                    109M     0  109M   0% /dev/shm
tmpfs                  109M     0  109M   0% /sys/fs/cgroup
/dev/mapper/vg0-data0  2.7G  628M  2.0G  25% /data

root@e286573cd16186:/usr/local/bin# fallocate -l 1800M /media/disk3.img
```

Reference [guide on how to create loop device](https://linuxconfig.org/how-to-create-loop-devices-on-linux).

```shell
root@e286573cd16186:/usr/local/bin# losetup -f
/dev/loop3
root@e286573cd16186:/usr/local/bin# losetup /dev/loop3 /media/disk3.img

# follow re-creating disk steps from above, pvcreate + vgextend
# now I remember to resize, I let `lvextend` detect FS type & resize
root@e286573cd16186:/usr/local/bin# lvextend -L 4G /dev/mapper/vg0-data0 -A y --resizefs
  Size of logical volume vg0/data0 unchanged from 4.00 GiB (1024 extents).
  Logical volume vg0/data0 successfully resized.
resize2fs 1.45.5 (07-Jan-2020)
Filesystem at /dev/mapper/vg0-data0 is mounted on /data; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mapper/vg0-data0 is now 1048576 (4k) blocks long.
```

> Try to pretend like this is a production system and minimize any downtime on the HTTP object service. The notes you're keeping are the sort of raw context we'd want to have if we were doing a retrospective after an outage.

## Script to autoresize

This script will need to be run in the background, `./autoresize.sh &`. This can be set up as a systemd unit or otherwise.

```shell
#!/bin/bash

while true; do
        df -H | grep data | awk '{ print $5 }' | cut -d'%' -f1 | while read -r usage;
        do
                if [[ $usage -ge 99 ]]; then
                        echo "$(date) Usage: $(df -h | grep data), increasing 500MB" >> /var/log/autoresize.log
                        lvextend -L +500M /dev/mapper/vg0-data0 -A y --resizefs >> /var/log/autoresize.log 2>&1

                        if [[ $? -ne 0 ]]; then
                                echo "Failed to extend LV" >> /var/log/autoresize.log
                                # don't want to keep printing logs forever, but exiting on failure makes this brittle
                                # at this point probably ping someone to look at this
                                exit 1 
                        fi
                fi
        done
done
```



## test-storage.sh output

```shell
root@e286573cd16186:/# test-storage.sh
# Ensuring we can write to the data dir...
PASS: Successfully wrote to the data dir.

# Verifying there are 4 PVs in the vg0 VG...
PASS: Detected 4 PVs

# Verifying that the vg0 VG is built from 4 1.8G PVs
PASS: All PVs are correctly sized.

# Checking that the data0 volume is 4G...
PASS: The data0 LV is 4096 MiB.

# Ensuring we don't have any corrupted objects...
PASS: Data in /data/objects checksums correctly.

# Simulating object creation to trigger disk growth...
100M file created: /data/test/1 ( 18% disk used)
100M file created: /data/test/2 ( 21% disk used)
100M file created: /data/test/3 ( 23% disk used)
100M file created: /data/test/4 ( 26% disk used)
100M file created: /data/test/5 ( 28% disk used)
100M file created: /data/test/6 ( 31% disk used)
100M file created: /data/test/7 ( 34% disk used)
100M file created: /data/test/8 ( 36% disk used)
100M file created: /data/test/9 ( 39% disk used)
100M file created: /data/test/10 ( 41% disk used)
100M file created: /data/test/11 ( 44% disk used)
100M file created: /data/test/12 ( 46% disk used)
100M file created: /data/test/13 ( 49% disk used)
100M file created: /data/test/14 ( 51% disk used)
100M file created: /data/test/15 ( 54% disk used)
100M file created: /data/test/16 ( 56% disk used)
100M file created: /data/test/17 ( 59% disk used)
100M file created: /data/test/18 ( 61% disk used)
100M file created: /data/test/19 ( 64% disk used)
100M file created: /data/test/20 ( 66% disk used)
100M file created: /data/test/21 ( 69% disk used)
100M file created: /data/test/22 ( 71% disk used)
100M file created: /data/test/23 ( 74% disk used)
100M file created: /data/test/24 ( 76% disk used)
100M file created: /data/test/25 ( 79% disk used)
100M file created: /data/test/26 ( 82% disk used)
100M file created: /data/test/27 ( 84% disk used)
100M file created: /data/test/28 ( 87% disk used)
100M file created: /data/test/29 ( 89% disk used)
100M file created: /data/test/30 ( 92% disk used)
100M file created: /data/test/31 ( 94% disk used)
100M file created: /data/test/32 ( 86% disk used)
100M file created: /data/test/33 ( 88% disk used)
100M file created: /data/test/34 ( 90% disk used)
100M file created: /data/test/35 ( 93% disk used)
100M file created: /data/test/36 ( 95% disk used)
100M file created: /data/test/37 ( 88% disk used)
100M file created: /data/test/38 ( 90% disk used)
100M file created: /data/test/39 ( 92% disk used)
100M file created: /data/test/40 ( 94% disk used)
100M file created: /data/test/41 ( 87% disk used)
100M file created: /data/test/42 ( 89% disk used)
100M file created: /data/test/43 ( 91% disk used)
100M file created: /data/test/44 ( 93% disk used)
100M file created: /data/test/45 ( 94% disk used)
100M file created: /data/test/46 ( 88% disk used)
100M file created: /data/test/47 ( 90% disk used)
100M file created: /data/test/48 ( 92% disk used)
100M file created: /data/test/49 ( 93% disk used)
100M file created: /data/test/50 ( 95% disk used)
100M file created: /data/test/51 ( 89% disk used)
100M file created: /data/test/52 ( 91% disk used)
100M file created: /data/test/53 ( 92% disk used)
100M file created: /data/test/54 ( 94% disk used)
100M file created: /data/test/55 ( 89% disk used)
100M file created: /data/test/56 ( 90% disk used)
100M file created: /data/test/57 ( 92% disk used)
100M file created: /data/test/58 ( 93% disk used)
100M file created: /data/test/59 ( 94% disk used)
100M file created: /data/test/60 ( 96% disk used)
100M file created: /data/test/61 ( 97% disk used)
100M file created: /data/test/62 ( 99% disk used)

# Checking that the /data filesystem is at least 6G now...
PASS: /data is at least 6G

## LVM Summary
  --- Physical volume ---
  PV Name               /dev/loop0
  VG Name               vg0
  PV Size               <1.76 GiB / not usable 4.00 MiB
  Allocatable           NO
  PE Size               4.00 MiB
  Total PE              449
  Free PE               0
  Allocated PE          449
  PV UUID               yrcDAb-MOZI-IE0E-c55z-UOMm-r3iy-KVTTRD

  --- Physical volume ---
  PV Name               /dev/loop2
  VG Name               vg0
  PV Size               <1.76 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              449
  Free PE               0
  Allocated PE          449
  PV UUID               W1Be1D-SXBS-cgJr-YQkz-D2Vi-PsvD-kJuFl4

  --- Physical volume ---
  PV Name               /dev/loop1
  VG Name               vg0
  PV Size               <1.76 GiB / not usable 4.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              449
  Free PE               0
  Allocated PE          449
  PV UUID               3AZdKo-sZ98-jpWc-7vNa-UTcN-P8aZ-J9Q6Yc

  --- Physical volume ---
  PV Name               /dev/loop3
  VG Name               vg0
  PV Size               <1.76 GiB / not usable 4.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              449
  Free PE               22
  Allocated PE          427
  PV UUID               0rq4vn-jEdx-0F0G-oPDs-PNSP-V7ew-y54mEw

  --- Volume group ---
  VG Name               vg0
  System ID
  Format                lvm2
  Metadata Areas        4
  Metadata Sequence No  18
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                4
  Act PV                4
  VG Size               <7.02 GiB
  PE Size               4.00 MiB
  Total PE              1796
  Alloc PE / Size       1774 / <6.93 GiB
  Free  PE / Size       22 / 88.00 MiB
  VG UUID               C2cFrL-305n-uXVS-1C86-0OPm-B6yY-j4tzMO

  --- Logical volume ---
  LV Path                /dev/vg0/data0
  LV Name                data0
  VG Name                vg0
  LV UUID                zEZch3-vGY0-Immv-E2PO-zMrd-3jHy-yDtD8i
  LV Write Access        read/write
  LV Creation host, time e286573cd16186, 2024-04-18 23:09:34 +0000
  LV Status              available
  # open                 1
  LV Size                <6.93 GiB
  Current LE             1774
  Segments               4
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

  --- Segments ---
  Logical extents 0 to 448:
    Type                linear
    Physical volume     /dev/loop0
    Physical extents    0 to 448

  Logical extents 449 to 897:
    Type                linear
    Physical volume     /dev/loop2
    Physical extents    0 to 448

  Logical extents 898 to 1346:
    Type                linear
    Physical volume     /dev/loop1
    Physical extents    0 to 448

  Logical extents 1347 to 1773:
    Type                linear
    Physical volume     /dev/loop3
    Physical extents    0 to 426
```

## Setting loop0 to allocatable

I noticed this last before submitting. Googling was not clear to me in what situations where a PV is considered "not allocatable", so I just changed the attribute.

```shell
root@e286573cd16186:/home# pvchange --allocatable y /dev/loop0
  Physical volume "/dev/loop0" changed
  1 physical volume changed / 0 physical volumes not changed
```