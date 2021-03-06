<!---
   Copyright 2014 Portland State University

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
--->

Lab 5: Storage
===================

In this lab, you will be tasked with creating filesystems on Solaris and Linux.

Section 1: ZFS
--------------

Okay, Welcome all of y'all to the most important event of your entire life, ever.
ZFS once stood for Zetabyte File System, but no longer means anything anymore.
We (and a lot of other companies) use ZFS for our most critical File system needs.
The best part of ZFS is that it is all software defined storage, and highly configurable.
We can define what we want wit code, and ZFS sees to it that it will be done.

### Format

#### Figure out which disks are on the system

```bash
bunny:/# format
AVAILABLE DISK SELECTIONS:
    0. c10t0d0 <ATA-HITACHI HDS7250S-AV0A-465.76GB>
        /pci@1,0/pci1022,7458@4/pci11ab,11ab@1/disk@0,0
        /dev/chassis/SYS/HD_ID_0/disk
    1. c10t4d0 <ATA-HITACHI HDS7250S-AV0A-465.76GB>
        /pci@1,0/pci1022,7458@4/pci11ab,11ab@1/disk@4,0
        /dev/chassis/SYS/HD_ID_1/disk
```

### zpool
We start everything with zpool. zpool takes a list of locations, that it will pool together (heh)
and later define filesystems on top of (with zfs). Lets's take a look:

#### create a zpool

So to make a pool, you need three things, a name, a pooling strategy, and
a list of devices to pool together.

* Name
    * The name of the pool
* Pool Strategy
    * How you want to arrange the 'disks' together
* Devices
    * This can be 'real' devices, or even pseudo devices (like loopback files)

```sh
bunny:/racoonSys# ls
file1   file10  file2   file3   file4   file5   file6   file7   file8   file9

bunny:/racoonSys# du -h ./*
    64M   ./file1
    64M   ./file10
    64M   ./file2
    64M   ./file3
    64M   ./file4
    64M   ./file5
    64M   ./file6
    64M   ./file7
    64M   ./file8
    64M   ./file9

bunny:/racoonSys# zpool create racoonSys2 /racoonSys/file{1,2,3,4,5,6,7,8,9,10}

bunny:/racoonSys# zpool list
    NAME         SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
    racoonSys   9.94G   716M  9.24G   7%  1.00x  ONLINE  -
    racoonSys2   590M   116K   590M   0%  1.00x  ONLINE  -
    rpool        464G  15.4G   449G   3%  1.00x  ONLINE  -

bunny:/racoonSys# zpool status racoonSys2
  pool: racoonSys2
  state: ONLINE
  scan: none requested
  config:

    NAME                 STATE     READ WRITE CKSUM
    racoonSys2           ONLINE       0     0     0
      /racoonSys/file1   ONLINE       0     0     0
      /racoonSys/file2   ONLINE       0     0     0
      /racoonSys/file3   ONLINE       0     0     0
      /racoonSys/file4   ONLINE       0     0     0
      /racoonSys/file5   ONLINE       0     0     0
      /racoonSys/file6   ONLINE       0     0     0
      /racoonSys/file7   ONLINE       0     0     0
      /racoonSys/file8   ONLINE       0     0     0
      /racoonSys/file9   ONLINE       0     0     0
      /racoonSys/file10  ONLINE       0     0     0

  errors: No known data errors

bunny:/racoonSys#
```

#### verify the pool
`zpool list` is our quick and dirty way for looking at all of the pools
defined on a system with zfs enabled.

Note: rpool is the typical name of the root system on a solaris system.

```sh
bunny:/# zpool list
NAME         SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
bear        1.81T    97K  1.81T   0%  1.00x  ONLINE  -
bobcat      1.81T    97K  1.81T   0%  1.00x  ONLINE  -
cougar      1.81T  10.0G  1.80T   0%  1.00x  ONLINE  -
jaguar      1.81T    97K  1.81T   0%  1.00x  ONLINE  -
leapord     1.81T    97K  1.81T   0%  1.00x  ONLINE  -
lion        1.81T   150K  1.81T   0%  1.00x  ONLINE  -
panther     1.81T    97K  1.81T   0%  1.00x  ONLINE  -
racoonSys   9.94G   716M  9.24G   7%  1.00x  ONLINE  -
racoonSys2   592M   182K   592M   0%  1.00x  ONLINE  -
rpool        464G  15.4G   449G   3%  1.00x  ONLINE  -
tiger       1.36T    94K  1.36T   0%  1.00x  ONLINE  -

bunny:/#
```

#### detailed status of a pool
`zpool status` can give you detailed information about a pool at a time. If you don't give it any arguments, it will give you status reports for all of the zpools. You can see here what devices are in what raid configurations, as well as the health of the pool and any errors in the pool.

```sh
bunny:~# zpool status racoonSys2
  pool: racoonSys2
 state: ONLINE
  scan: none requested
config:

        NAME                 STATE     READ WRITE CKSUM
        racoonSys2           ONLINE       0     0     0
          /racoonSys/file1   ONLINE       0     0     0
          /racoonSys/file2   ONLINE       0     0     0
          /racoonSys/file3   ONLINE       0     0     0
          /racoonSys/file4   ONLINE       0     0     0
          /racoonSys/file5   ONLINE       0     0     0
          /racoonSys/file6   ONLINE       0     0     0
          /racoonSys/file7   ONLINE       0     0     0
          /racoonSys/file8   ONLINE       0     0     0
          /racoonSys/file9   ONLINE       0     0     0
          /racoonSys/file10  ONLINE       0     0     0

errors: No known data errors
bunny:~#
```

#### destroy the pool
Exactly as it sounds... This will destroy that pool you worked so long to configure.
Easy to do too! And POOF... Gone. See yeah forever! And it wont even call you back, after its done.

```sh
bunny:/# zpool destroy racoonSys2

bunny:/#
```

#### Now make that zpool again and lets move on


### zfs

#### Create a zfs filesystem

```bash
zfs create tank/media
```

#### Enable zfs snapshots

```bash
# TODO: wren plz pre-install package so they don't have to wait for ages
# svcs -av  | grep snap
disabled       -             Dec_19           - svc:/system/filesystem/zfs/auto-snapshot:frequent
disabled       -             Dec_19           - svc:/system/filesystem/zfs/auto-snapshot:hourly
disabled       -             Dec_19           - svc:/system/filesystem/zfs/auto-snapshot:monthly
disabled       -             Dec_19           - svc:/system/filesystem/zfs/auto-snapshot:weekly
disabled       -             Dec_19           - svc:/system/filesystem/zfs/auto-snapshot:daily
```

#### Set the refquota to 5G

```bash
zfs set refquota=5G tank/media
```

#### Create a user janaka and a group cats


```
# TODO, or should this be in vagrant already?
```

#### Set the owner of the the filesystem to janaka and the group to cats

```bash
chown $user:$group /tank/media
```

#### Set the permissions to owner/group rwx with guid bit

```
chmod 2770 /tank/media
```

#### Enable NFS


```bash
zfs set share.nfs=on tank/media
```

#### Restrict to the linux client IP addr

```bash
zfs set share.nfs.sec.default.rw=$CLIENT_IP_ADDR tank/media
```
#### Mount /tank/media on the Linux client

#### mount

```
# TODO
mount
```

#### Verify user janaka can read/write

#### Verify users in group cat can read/write

#### Verify users not in group cat cannot read/write

### zfs send and receive: Send a whole file system to another computer!

#### doing it the first time
You can use zfs send and zfs receive to copy file systems on zfs from one computer to another. In order to do this, you will need a snapshot; either use an existing snapshot, or create one with `zfs snapshot`.

When you use zfs send and receive, be careful of mountpoints and names; if a file system already exists with the exact same name as the one you're about to send, it will be overwritten, and if two file systems have the same mount point, they'll fight over it.

The basic zfs send/recv command (if at first you don't succeed, look into flags) looks like this:
`zfs send racoonSys2@test | ssh ${other_host} zfs receive leopardSys2`

Note: It's possible that the new file system created on the other machine won't mount automatically, in which case you will need to use `zfs mount -a`.

#### Incremental snapshots
Now that you've sent an initial snapshot of the file system, make some changes to a file in that file system (touch a new file, etc) and then create another snapshot (this part of the process is identical). Try to name the new snapshot something that indicates that it came chronologically after the first snapshot.

Now, since you have two different snapshots, you can send an incremental change to the other computer! This is faster than sending the whole file system again, since you only need to send the things that have changed since you sent a snapshot.
`zfs send -i test racoonSys2@test_the_second | ssh ${other_host} zfs receive leopardSys2`



Logical Volume Manager
----------------------

### Volume Groups

All of the commands that deal with Volume groups are prefixed with the letters
'vg' thus, if your tab-completion is awesome, you can vg<tab><tab> to get a quick
and dirty look at all of the handy commands.

```sh
-> % vg
vgcfgbackup    vgchange       vgconvert      vgdb           vgexport
vgimport       vgmerge        vgreduce       vgrename       vgscan
vgcfgrestore   vgck           vgcreate       vgdisplay      vgextend
vgimportclone  vgmknodes      vgremove       vgs            vgsplit
```

#### Create a Volume Group
#### List your Volume Groups
#### Delete a Volume Group

### Logical Volumes
```sh
-> % lv
lvchange     lvcreate     lvextend     lvmchange    lvmdiskscan
lvmetad      lvmsar       lvremove     lvresize     lvscan
lvconvert    lvdisplay    lvm          lvmconf      lvmdump
lvmsadc      lvreduce     lvrename     lvs
```
#### Create a Logical Volume
#### List your Delete a Logical Volume
#### Resize the Volume

### Partitioning

#### Create a ext3 filestystem on the first partition
#### Create a ext4 filestystem on the second partition


Advanced Section
================

### dd
dd is the swiss army tool of file-data, we can use dd to create 'fake' block devices
that can be mounted and used elsewhere.

lets create a 1G file that we can use to turn into a (fake) filesystem:

`dd if=/dev/zero of=/destination/file bs=1 count=0 seek=1G`

This command should execute rather quickly. We are using a trick with seek, to tell
the system how big it should be without actually going and taking out all of the space.

### losetup

now you can take the file you just made and associate it with a loopback device.
In layman's terms, it takes the file and maks a block device out of it.
...errr it makes the file look like a hard drive...

`losetup /dev/loop0 /destination/file`


### mkfs.*
You can then make a filesystem on that (fake) device by using mkfs.
mkfs comes with a few built in filesystem types that you can specify after the '.'

let's make an ext4 system on our (fake hdd)

```sh
[root@host ~]# mkfs.ext4 /dev/loop0
mke2fs 1.42.8 (20-Jun-2013)
Discarding device blocks: done
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
32768 inodes, 131072 blocks
6553 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=134217728
4 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304
        
        Allocating group tables: done
        Writing inode tables: done
        Creating journal (4096 blocks): done
        Writing superblocks and filesystem accounting information: done
```


### lsblk
lsblk is a quick and beautiful tool for looking at the filesystem tree.

```sh

[root@fatdadd mnt]# lsblk
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda       8:0    0 149.1G  0 disk
├─sda1    8:1    0  46.6G  0 part /
└─sda2    8:2    0 102.5G  0 part /home
loop0     7:0    0   512M  0 loop /mnt
mmcblk0 179:0    0  59.8G  0 disk
```

here is more exapmle output from a more interesting setup.
```sh
-> % lsblk
NAME                                 MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0                                  7:0    0  24.4M  0 loop /srv/node/1
loop1                                  7:1    0  24.4M  0 loop /srv/node/2
sr0                                   11:0    1  1024M  0 rom
sda                                    8:0    0  84.7G  0 disk
├─sda1                                 8:1    0   731M  0 part /boot
├─sda2                                 8:2    0     1K  0 part
└─sda5                                 8:5    0    84G  0 part
  ├─openstack2--vg-root (dm-0)       252:0    0  19.1G  0 lvm  /
  ├─openstack2--vg-var (dm-1)        252:1    0   5.7G  0 lvm  /var
  ├─openstack2--vg-swap_1 (dm-2)     252:2    0   3.8G  0 lvm  [SWAP]
  ├─openstack2--vg-tmp (dm-3)        252:3    0   2.9G  0 lvm  /tmp
  └─openstack2--vg-disk+trump (dm-4) 252:4    0  52.4G  0 lvm  /disk/trump
```

### format
solaris command for determining disks that the kernel can see.

```sh
chandra:~# format
Searching for disks...done


AVAILABLE DISK SELECTIONS:
       0. c1t0d0 <SUN146G cyl 14087 alt 2 hd 24 sec 848>
            /pci@0/pci@0/pci@2/scsi@0/sd@0,0
       1. c1t1d0 <SUN146G cyl 14087 alt 2 hd 24 sec 848>
            /pci@0/pci@0/pci@2/scsi@0/sd@1,0
    Specify disk (enter its number): ctrl-c
```
to do a quick and dirty lookup of the disks (and their names)

### fdisk/cfdisk

These tools are used for creating partitions within a filesystem.

```sh
-> % sudo fdisk -l /dev/sda

Disk /dev/sda: 149.1 GiB, 160041885696 bytes, 312581808 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4b2bb6c8

Device    Boot     Start       End    Blocks  Id System
/dev/sda1           2048  97658879  48828416  83 Linux
/dev/sda2       97658880 312581807 107461464  83 Linux
```


RAID
----

### What is Raid?

RAID (Redundant Array of Inexpensive Disks) is the ability to combine multiple disks
to attain greater speed/redundancy/both from disk usage. After you RAID together multiple
drives, you then go on and treat them as if the sum total is one drive.

#### Levels of RAID

`RAID 0`

* Striping
* Spread the data out over multiple disks
* Allows for parallel reads on the same file

`RAID 1`

* Mirroring
* Creates redundancy, or, makes multiple copies of a file over all disks
* If a disk goes down, you laugh, and say "But RAID 1"

`RAID 5`

* Block Level striping w/ parity
* The file is distributed accross multiple drives, along with parity bits
  that allow the information of a disk to be recalulated in the event of failure
* Requires at least 3 disks
* Allows single disk failure

`RAID 10`

* Combination pizza hut and taco bell
* Or in technical terms, both RAID 1 and RAID 0 at the same time
* Example, with four disks, mirror first two, mirror last two and then strip the result

#### Hardware vs Software

You have two choices in how you do this thing. Either you buy a dedicated card, that does the
RAID and keeps track of what's going on. OR, you create a software RAID and let the OS keep
track of what to do.

### mdadm
Linux utility for creating software RAIDs

#### RAID Reconnaissance

This file will show you the current status of your local RAID.
It's jam packed full of good info that you will need for debugging and
referencing your RAID setup.

This example shows a system with a RAID0 and is using two devices `sda` and `sdb`
```sh
-> % cat /proc/mdstat
Personalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10]
md2 : active raid1 sda[0] sdb[1]
      976631360 blocks super 1.2 [2/2] [UU]
      unused devices: <none> </none>
```

Now that we know the names of the raids used on the system, we can get more details.

`mdadm --detail $RAIDNAME`

```sh
root@russell:~# mdadm --detail /dev/md2
    /dev/md2:
        Version : 1.2
        Creation Time : Sat Apr 27 19:16:17 2013
            Raid Level : raid1
            Array Size : 976631360 (931.39 GiB 1000.07 GB)
        Used Dev Size : 976631360 (931.39 GiB 1000.07 GB)
            Raid Devices : 2
            Total Devices : 2
            Persistence : Superblock is persistent

            Update Time : Fri Jan 24 00:15:48 2014
            State : clean
            Active Devices : 2
            Working Devices : 2
            Failed Devices : 0
            Spare Devices : 0

            Name : ubuntu:2
            UUID : 4032b613:380ebddb:db9a61b0:42f8680a
            Events : 97

            Number   Major   Minor   RaidDevice State
            0       8        0        0      active sync   /dev/sda
            1       8       16        1      active sync   /dev/sdb
```

#### Creating a RAID

That's all fine and dandy. But how make a new RAID? Say... The fabled RAID 10? The hard way?

Let's make some fake disks really quick... How about Four 1G disks?

```sh
[root@fatdadd disks]# for i in `seq 4`; do dd if=/dev/zero of=./hdd$i bs=1M count=1000; done
1000+0 records in
1000+0 records out
1048576000 bytes (1.0 GB) copied, 0.637248 s, 1.6 GB/s
1000+0 records in
1000+0 records out
1048576000 bytes (1.0 GB) copied, 0.572313 s, 1.8 GB/s
1000+0 records in
1000+0 records out
1048576000 bytes (1.0 GB) copied, 3.87051 s, 271 MB/s
1000+0 records in
1000+0 records out
1048576000 bytes (1.0 GB) copied, 5.8634 s, 179 MB/s
```
Now lets mount them as loop devices

```sh
[root@fatdadd disks]# for i in `seq 4`;do losetup /dev/loop$i ./hdd$i; done

[root@fatdadd disks]# losetup
NAME       SIZELIMIT OFFSET AUTOCLEAR RO BACK-FILE
/dev/loop1         0      0         0  0 /root/disks/hdd1
/dev/loop2         0      0         0  0 /root/disks/hdd2
/dev/loop3         0      0         0  0 /root/disks/hdd3
/dev/loop4         0      0         0  0 /root/disks/hdd4
```

Awesome! Now we do the dirty... Raid the first two and call it `/dev/md1`

```sh
[root@fatdadd disks]# mdadm --create /dev/md1 --level=stripe --raid-devices=2 /dev/loop{1,2}
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.

[root@fatdadd proc]# cat /proc/mdstat
Personalities : [raid0]
md1 : active raid0 loop2[1] loop1[0]
      2045952 blocks super 1.2 512k chunks

unused devices: <none>
```
One down, Now lets make `/dev/md2` with the other two loop devices

```sh
[root@fatdadd proc]# mdadm --create /dev/md2 --level=stripe --raid-devices=2 /dev/loop{3,4}
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md2 started.

[root@fatdadd proc]# cat /proc/mdstat
Personalities : [raid0]
md2 : active raid0 loop4[1] loop3[0]
      2045952 blocks super 1.2 512k chunks

md1 : active raid0 loop2[1] loop1[0]
      2045952 blocks super 1.2 512k chunks

unused devices: <none>
```

Okay! Here we go, Now we are going to zip those two md devices into one mirrored device we'll call `/dev/md0`

```sh
[root@fatdadd proc]# mdadm --create /dev/md0 --level=mirror --raid-devices=2 /dev/md{1,2}
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

[root@fatdadd proc]# cat /proc/mdstat
Personalities : [raid0] [raid1]
md0 : active raid1 md2[1] md1[0]
      2044928 blocks super 1.2 [2/2] [UU]
      [========>............]  resync = 43.7% (895104/2044928) finish=0.1min speed=127872K/sec

md2 : active raid0 loop4[1] loop3[0]
      2045952 blocks super 1.2 512k chunks

md1 : active raid0 loop2[1] loop1[0]
      2045952 blocks super 1.2 512k chunks

unused devices: <none>
```

YEEEEEEEEEEE!!!!!!! We did it!

Now throw an ext4 filesystem on `/dev/md0` and then mount that sucker to `/mnt`

```sh
[root@fatdadd /]# mkfs.ext4 /dev/md0
mke2fs 1.42.8 (20-Jun-2013)
Discarding device blocks: done
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=128 blocks, Stripe width=256 blocks
128000 inodes, 511232 blocks
25561 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=524288000
16 block groups
32768 blocks per group, 32768 fragments per group
8000 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

[root@fatdadd /]# mount /dev/md0 /mnt

[root@fatdadd /]# cd /mnt

[root@fatdadd mnt]# ls
lost+found

[root@fatdadd mnt]# df -h .
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0        1.9G  3.0M  1.8G   1% /mnt

[root@fatdadd mnt]# mdadm --detail /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Sat Feb  1 17:24:14 2014
     Raid Level : raid1
     Array Size : 2044928 (1997.34 MiB 2094.01 MB)
  Used Dev Size : 2044928 (1997.34 MiB 2094.01 MB)
   Raid Devices : 2
  Total Devices : 2
    Persistence : Superblock is persistent

    Update Time : Sat Feb  1 17:25:52 2014
          State : clean
 Active Devices : 2
Working Devices : 2
 Failed Devices : 0
  Spare Devices : 0

           Name : fatdadd:0  (local to host fatdadd)
           UUID : c08744c0:7c0e07df:03d8395e:00b0dd11
         Events : 17

    Number   Major   Minor   RaidDevice State
       0       9        1        0      active sync   /dev/md1
       1       9        2        1      active sync   /dev/md2

```

Great! Now how about a shortcut? We've used `--level=stripe` and `--level=mirror`, but
what if I told you they had synonyms? `--level=0` and `--level=1` respectively? And then
What if then proceeded to explain the existence of `--level=10`? Would that smash you mind
to bits?

Well tell the people around you how sorry you are about to be once you shower them with
pieces of cerebrum and skull cause `--level=10` is in fact a thing. Tell your mom.

Okay, Lets disassemble the Raid we have going now and use the loop devices. (don't forget
to unmount it from `/mnt`)

```sh
[root@fatdadd mnt]# mdadm --misc -S /dev/md0
mdadm: Cannot get exclusive access to /dev/md0:Perhaps a running process, mounted filesystem or active volume group?

[root@fatdadd mnt]# cd /

[root@fatdadd /]# umount /dev/md0

[root@fatdadd /]# mdadm --misc -S /dev/md0
mdadm: stopped /dev/md0

[root@fatdadd /]# cat /proc/mdstat
Personalities : [raid0] [raid1]
md2 : active raid0 loop4[1] loop3[0]
      2045952 blocks super 1.2 512k chunks

md1 : active raid0 loop2[1] loop1[0]
      2045952 blocks super 1.2 512k chunks

unused devices: <none>
[]
</none>
```

And now take down the two stripes

```sh
[root@fatdadd /]# mdadm --misc -S /dev/md1
mdadm: stopped /dev/md1

[root@fatdadd /]# mdadm --misc -S /dev/md2
mdadm: stopped /dev/md2

[root@fatdadd /]# cat /proc/mdstat
Personalities : [raid0] [raid1]
unused devices: <none></none>
```

Okay ladies and Gentlemen, Let's F*cking do this sh*t

```sh
[root@fatdadd ~]# mdadm --create /dev/md0 --level=10 --raid-devices=4 /dev/loop{1,2,3,4}
mdadm: /dev/loop1 appears to be part of a raid array:
       level=raid0 devices=2 ctime=Sat Feb  1 17:23:17 2014
mdadm: /dev/loop2 appears to be part of a raid array:
       level=raid0 devices=2 ctime=Sat Feb  1 17:23:17 2014
mdadm: /dev/loop3 appears to be part of a raid array:
       level=raid0 devices=2 ctime=Sat Feb  1 17:23:43 2014
mdadm: /dev/loop4 appears to be part of a raid array:
       level=raid0 devices=2 ctime=Sat Feb  1 17:23:43 2014
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

[root@fatdadd ~]# cat /proc/mdstat
Personalities : [raid0] [raid1] [raid10]
md0 : active raid10 loop4[3] loop3[2] loop2[1] loop1[0]
      2045952 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]
      [===============>.....]  resync = 76.4% (1564992/2045952) finish=0.0min speed=223570K/sec

unused devices: <none></none>
```

Pick your Jaw up, print this out and use it as wrapping paper for your mothers day present homie.

Now... Go do [Advanced Things with RAID](mdadm-hotspare.md)

