# MDADM_otus
 
1. Создал рабочую папку mdadm, загрузил в нее Vagrantfile

2. В Vargantfile изменил настройки сетевого интерфейса, добавил 5 диск

3. Выполнил vagrant up и подключился к ВМ.

3*. Т.к. Centos перестала поддерживаться и из коробки Vagrantfile утилита mdadm не установлена, пришлось добавить зеркала репозиториев, чтобы установить утилиту mdadm для создания программного RAID по выполнению условия ДЗ

Выполнил следующие команды:

[root@otuslinux ~]# sudo sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
[root@otuslinux ~]# sudo sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
[root@otuslinux ~]# sudo yum update -y
_______________________________________________________
После успешного обновления ОС, установил утилиту mdadm:

[root@otuslinux ~]# yum install -y mdadm smartmontools hdparm gdisk
Failed to set locale, defaulting to C.UTF-8
Repository extras is listed more than once in the configuration
Last metadata expiration check: 0:11:11 ago on Thu Oct 31 12:25:39 2024.
Package hdparm-9.54-2.el8.x86_64 is already installed.
Dependencies resolved.
===================================================================================================================================================================
 Package                                    Architecture                        Version                                  Repository                           Size
===================================================================================================================================================================
Installing:
 gdisk                                      x86_64                              1.0.3-6.el8                              baseos                              240 k
 mdadm                                      x86_64                              4.2-rc2.el8                              baseos                              460 k
 smartmontools                              x86_64                              1:7.1-1.el8                              baseos                              544 k
Upgrading:
 hdparm                                     x86_64                              9.54-4.el8                               baseos                              100 k

Transaction Summary
===================================================================================================================================================================
Install  3 Packages
Upgrade  1 Package
________________________________
Далее выполнил шаги из методички


4. Выполнил команду sudo lshw -short | grep disk, получил список имеющихся дисков в ВМ
/0/100/1.1/0.0.0    /dev/sda   disk        42GB VBOX HARDDISK
/0/100/d/0          /dev/sdb   disk        262MB VBOX HARDDISK
/0/100/d/1          /dev/sdc   disk        262MB VBOX HARDDISK
/0/100/d/2          /dev/sdd   disk        262MB VBOX HARDDISK
/0/100/d/3          /dev/sde   disk        262MB VBOX HARDDISK

5. Выполнил команду: sudo fdisk -l
[root@otuslinux ~]# sudo fdisk -l
Disk /dev/sda: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xef431952

Device     Boot Start      End  Sectors Size Id Type
/dev/sda1  *     2048 20971519 20969472  10G 83 Linux


Disk /dev/sdb: 250 MiB, 262144000 bytes, 512000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 250 MiB, 262144000 bytes, 512000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdd: 250 MiB, 262144000 bytes, 512000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sde: 250 MiB, 262144000 bytes, 512000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdf: 250 MiB, 262144000 bytes, 512000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

6. Выполнил команду: mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
[root@otuslinux ~]# mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
mdadm: Unrecognised md component device - /dev/sdb
mdadm: Unrecognised md component device - /dev/sdc
mdadm: Unrecognised md component device - /dev/sdd
mdadm: Unrecognised md component device - /dev/sde
mdadm: Unrecognised md component device - /dev/sdf

7. Выполнил команду и создал RAID 5: mdadm --create --verbose /dev/md0 -l 5 -n 5 /dev/sd{b,c,d,e,f}
[root@otuslinux ~]# mdadm --create --verbose /dev/md0 -l 5 -n 5 /dev/sd{b,c,d,e,f}
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 253952K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

8. Проверил что RAID собрался нормально:
[root@otuslinux ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sdf[5] sde[3] sdd[2] sdc[1] sdb[0]
      1015808 blocks super 1.2 level 5, 512k chunk, algorithm 2 [5/5] [UUUUU]

unused devices: <none>
_______________________

[root@otuslinux ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu Oct 31 12:42:04 2024
        Raid Level : raid5
        Array Size : 1015808 (992.00 MiB 1040.19 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Thu Oct 31 12:42:12 2024
             State : clean
    Active Devices : 5
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : 6324ffeb:0fee8ddb:f2e0ae27:00b60394
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       5       8       80        4      active sync   /dev/sdf

9. Создал конфигурационный файл mdadm.conf

С начала убедился, что информация верна:

[root@otuslinux ~]# mdadm --detail --scan --verbose
ARRAY /dev/md0 level=raid5 num-devices=5 metadata=1.2 name=otuslinux:0 UUID=6324ffeb:0fee8ddb:f2e0ae27:00b60394
   devices=/dev/sdb,/dev/sdc,/dev/sdd,/dev/sde,/dev/sdf
__________________
Создал mdadm.conf:

[root@otuslinux ~]# echo "DEVICE partitions" > /etc/mdadm/mdadm.conf

[root@otuslinux ~]# mdadm --detail --scan --verbose | awk '/ARRAY/ {print}'ARRAY /dev/md0 level=raid5 num-devices=5 metadata=1.2 name=otuslinux:0 UUID=6324ffeb:0fee8ddb:f2e0ae27:00b60394
awk: fatal: cannot open file `/dev/md0' for reading (Success)
[root@otuslinux ~]# mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf


10. Сломал/починил RAID

Искусственно “зафейлил” одно из блочных устройств командной:

[root@otuslinux ~]# mdadm /dev/md0 --fail /dev/sde
mdadm: set /dev/sde faulty in /dev/md0
_______________________________________
Посмотрел, как это отразилось на RAID:

[root@otuslinux ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sdf[5] sde[3](F) sdd[2] sdc[1] sdb[0]
      1015808 blocks super 1.2 level 5, 512k chunk, algorithm 2 [5/4] [UUU_U]

[root@otuslinux ~]# mdadm -D /dev/md0 | grep Devices
      Raid Devices : 5
     Total Devices : 5
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 1
     Spare Devices : 0
___________________________________
Удалил “сломанный” диск из массива:

[root@otuslinux ~]# mdadm /dev/md0 --remove /dev/sde
mdadm: hot removed /dev/sde from /dev/md0
_____________________
"Вставил" новый диск:
[root@otuslinux ~]# mdadm /dev/md0 --add /dev/sde
mdadm: added /dev/sde
__________________________
Посмотрел процесс ребилда:

[root@otuslinux ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sde[6] sdf[5] sdd[2] sdc[1] sdb[0]
      1015808 blocks super 1.2 level 5, 512k chunk, algorithm 2 [5/5] [UUUUU]

unused devices: <none>


11. Создал GPT раздел, пять партиций и смонтировал их на диск

Создал раздел GPT на RAID:

[root@otuslinux ~]# parted -s /dev/md0 mklabel gpt
_________________
Создал партиции:

[root@otuslinux ~]# parted /dev/md0 mkpart primary ext4 0% 20%
Information: You may need to update /etc/fstab.

[root@otuslinux ~]# parted /dev/md0 mkpart primary ext4 20% 40%
Information: You may need to update /etc/fstab.

[root@otuslinux ~]# parted /dev/md0 mkpart primary ext4 40% 60%
Information: You may need to update /etc/fstab.

[root@otuslinux ~]# parted /dev/md0 mkpart primary ext4 60% 80%
Information: You may need to update /etc/fstab.

[root@otuslinux ~]# parted /dev/md0 mkpart primary ext4 80% 100%
Information: You may need to update /etc/fstab.
________________________________________
Попробовал создать на этих партициях ФС:

[root@otuslinux ~]# for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 200704 1k blocks and 50200 inodes
Filesystem UUID: 924b3a9f-deb1-4a4c-840d-4c4171c51b0c
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 202752 1k blocks and 50800 inodes
Filesystem UUID: f998a36c-a7d8-414f-9358-dd75fab37121
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 204800 1k blocks and 51200 inodes
Filesystem UUID: da882d4f-2a2b-4197-a5a9-ab2a7ac9d030
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 202752 1k blocks and 50800 inodes
Filesystem UUID: 301411e1-56a0-48ac-a121-1d4afad61b8d
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 200704 1k blocks and 50200 inodes
Filesystem UUID: 7f63f63d-3be9-4efe-8152-450751406765
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
______________________________
И смонтировал их по каталогам:

[root@otuslinux ~]# mkdir -p /raid/part{1,2,3,4,5}
[root@otuslinux ~]# for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done