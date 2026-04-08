Скрипт для создания рейда:

#!/bin/sh
#create RAID

#view HDD

lsblk

#create RAID 6

sudo mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}

#control

cat /proc/mdstat




Отчет по командам для починки RAID и созданию разделов

Просмотр блочных устройств:


alex@ubuntuserver:~$ lsblk

NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   27G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0 11.5G  0 lvm  /
sdb                         8:16   0    1G  0 disk
sdc                         8:32   0    1G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sdf                         8:80   0    1G  0 disk
sdg                         8:96   0    1G  0 disk
sr0                        11:0    1 1024M  0 rom

Create RAID 6:

sudo mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}

mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 1046528K
mdadm: Defaulting to version 1.2 metadata

mdadm: array /dev/md0 started.

Control:

cat /proc/mdstat

Personalities : [raid0] [raid1] [raid4] [raid5] [raid6] [raid10] [linear]
md0 : active raid6 sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      3139584 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]

Control:

sudo mdadm -D /dev/md0

/dev/md0:
           Version : 1.2
     Creation Time : Wed Apr  8 11:07:14 2026
        Raid Level : raid6
        Array Size : 3139584 (2.99 GiB 3.21 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Wed Apr  8 11:07:25 2026
             State : clean
    Active Devices : 5
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : ubuntuserver:0  (local to host ubuntuserver)
              UUID : 4764a299:fe53e251:1e1989f7:bc6db9b0
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf

Make sdf fail:

sudo mdadm /dev/md0 --fail /dev/sdf

mdadm: set /dev/sdf faulty in /dev/md0


Control:

cat /proc/mdstat

Personalities : [raid0] [raid1] [raid4] [raid5] [raid6] [raid10] [linear]
md0 : active raid6 sdf[4](F) sde[3] sdd[2] sdc[1] sdb[0]
      3139584 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/4] [UUUU_]


Control:

sudo mdadm -D /dev/md0

/dev/md0:
           Version : 1.2
     Creation Time : Wed Apr  8 11:07:14 2026
        Raid Level : raid6
        Array Size : 3139584 (2.99 GiB 3.21 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Wed Apr  8 11:11:10 2026
             State : clean, degraded
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : ubuntuserver:0  (local to host ubuntuserver)
              UUID : 4764a299:fe53e251:1e1989f7:bc6db9b0
            Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       -       0        0        4      removed

       4       8       80        -      faulty   /dev/sdf

Remove sdf:

sudo mdadm /dev/md0 --remove /dev/sdf

mdadm: hot removed /dev/sdf from /dev/md0

Add new HDD:

sudo mdadm /dev/md0 --add /dev/sdg

mdadm: added /dev/sdg


Control:

cat /proc/mdstat

Personalities : [raid0] [raid1] [raid4] [raid5] [raid6] [raid10] [linear]
md0 : active raid6 sdg[5] sde[3] sdd[2] sdc[1] sdb[0]
      3139584 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]

Control:

sudo mdadm -D /dev/md0

/dev/md0:
           Version : 1.2
     Creation Time : Wed Apr  8 11:07:14 2026
        Raid Level : raid6
        Array Size : 3139584 (2.99 GiB 3.21 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Wed Apr  8 11:14:41 2026
             State : clean
    Active Devices : 5
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : ubuntuserver:0  (local to host ubuntuserver)
              UUID : 4764a299:fe53e251:1e1989f7:bc6db9b0
            Events : 39

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       5       8       96        4      active sync   /dev/sdg

Create GPT:

sudo parted -s /dev/md0 mklabel gpt


Create partitions:

sudo parted /dev/md0 mkpart primary ext4 0% 20%

Information: You may need to update /etc/fstab.

alex@ubuntuserver:~$ sudo parted /dev/md0 mkpart primary ext4 20% 40%

Information: You may need to update /etc/fstab.

alex@ubuntuserver:~$ sudo parted /dev/md0 mkpart primary ext4 40% 60%

Information: You may need to update /etc/fstab.

alex@ubuntuserver:~$ sudo parted /dev/md0 mkpart primary ext4 60% 80%

Information: You may need to update /etc/fstab.

alex@ubuntuserver:~$ sudo parted /dev/md0 mkpart primary ext4 80% 100%

Information: You may need to update /etc/fstab.

Create FileSystem:

for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done

Mont:

mkdir -p /raid/part{1,2,3,4,5}

for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done

