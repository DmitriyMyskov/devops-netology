# Домашнее задание к занятию "3.5. Файловые системы"

***
## Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.
###
Разреженный файл - файл, в котором последовательность нулевых байтов заменена на информацию об этих последовательностях.
Данное решение хорошо подходит для использования с виртуальными дисками на гостевой машине, файлов которые загружаются частями, а так же других крупных файлов.

***
## Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
###
Не могут, так как жесткие ссылки ссылаются на одну inode в которой и содержится информация о правах и владельце.
```bash
dmitriy@dmitriy-lin:~$ nano filetest.txt
dmitriy@dmitriy-lin:~$ nano filetest.md
dmitriy@dmitriy-lin:~$ ls -l |grep file
-rw-rw-r-- 1 dmitriy dmitriy   13 фев  8 11:08 filetest.md
-rw-rw-r-- 1 dmitriy dmitriy   10 фев  8 11:08 filetest.txt
dmitriy@dmitriy-lin:~$ ln filetest.md linkfiletest.md
dmitriy@dmitriy-lin:~$ ls -ilh |grep file
2795787 -rw-rw-r-- 2 dmitriy dmitriy   13 фев  8 11:08 filetest.md
2795786 -rw-rw-r-- 1 dmitriy dmitriy   10 фев  8 11:08 filetest.txt
2795787 -rw-rw-r-- 2 dmitriy dmitriy   13 фев  8 11:08 linkfiletest.md
dmitriy@dmitriy-lin:~$ chmod 0777 filetest.md 
dmitriy@dmitriy-lin:~$ ls -ilh |grep file
2795787 -rwxrwxrwx 2 dmitriy dmitriy   13 фев  8 11:08 filetest.md
2795786 -rw-rw-r-- 1 dmitriy dmitriy   10 фев  8 11:08 filetest.txt
2795787 -rwxrwxrwx 2 dmitriy dmitriy   13 фев  8 11:08 linkfiletest.md
```

***
## Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.
###
```bash
vagrant@vm-fs:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 61.9M  1 loop /snap/core20/1328
loop1                       7:1    0 67.2M  1 loop /snap/lxd/21835
loop2                       7:2    0 49.8M  1 loop /snap/snapd/17950
loop3                       7:3    0 63.3M  1 loop /snap/core20/1778
loop4                       7:4    0 91.9M  1 loop /snap/lxd/24061
sda                         8:0    0   64G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0  1.5G  0 part /boot
└─sda3                      8:3    0 62.5G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm  /
sdb                         8:16   0  2.5G  0 disk 
sdc                         8:32   0  2.5G  0 disk 
```

***
## Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.
###
```bash
vagrant@vm-fs:~$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xa904ee67.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-5242879, default 2048): 2048
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (4196352-5242879, default 4196352): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879): 

Created a new partition 2 of type 'Linux' and of size 511 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

vagrant@vm-fs:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 61.9M  1 loop /snap/core20/1328
loop1                       7:1    0 67.2M  1 loop /snap/lxd/21835
loop2                       7:2    0 49.8M  1 loop /snap/snapd/17950
loop3                       7:3    0 63.3M  1 loop /snap/core20/1778
loop4                       7:4    0 91.9M  1 loop /snap/lxd/24061
sda                         8:0    0   64G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0  1.5G  0 part /boot
└─sda3                      8:3    0 62.5G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm  /
sdb                         8:16   0  2.5G  0 disk 
├─sdb1                      8:17   0    2G  0 part 
└─sdb2                      8:18   0  511M  0 part 
sdc                         8:32   0  2.5G  0 disk 
```

***
## Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.
###
```bash
vagrant@vm-fs:~$ sudo sfdisk -d /dev/sdb | sudo sfdisk /dev/sdc
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0xa904ee67.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0xa904ee67

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
vagrant@vm-fs:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 61.9M  1 loop /snap/core20/1328
loop1                       7:1    0 67.2M  1 loop /snap/lxd/21835
loop2                       7:2    0 49.8M  1 loop /snap/snapd/17950
loop3                       7:3    0 63.3M  1 loop /snap/core20/1778
loop4                       7:4    0 91.9M  1 loop /snap/lxd/24061
sda                         8:0    0   64G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0  1.5G  0 part /boot
└─sda3                      8:3    0 62.5G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm  /
sdb                         8:16   0  2.5G  0 disk 
├─sdb1                      8:17   0    2G  0 part 
└─sdb2                      8:18   0  511M  0 part 
sdc                         8:32   0  2.5G  0 disk 
├─sdc1                      8:33   0    2G  0 part 
└─sdc2                      8:34   0  511M  0 part 
```

***
## Соберите `mdadm` RAID1 на паре разделов 2 Гб.
###
```bash
agrant@vm-fs:~$ sudo mdadm --create /dev/md1 --level=1 --raid-device=2 /dev/sd[bc]1
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
vagrant@vm-fs:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 63.3M  1 loop  /snap/core20/1778
loop1                       7:1    0 63.3M  1 loop  /snap/core20/1822
loop2                       7:2    0 91.9M  1 loop  /snap/lxd/24061
loop3                       7:3    0 67.2M  1 loop  /snap/lxd/21835
loop4                       7:4    0 49.8M  1 loop  /snap/snapd/17950
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0  1.5G  0 part  /boot
└─sda3                      8:3    0 62.5G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part
```

***
## Соберите `mdadm` RAID0 на второй паре маленьких разделов.
###
```bash
vagrant@vm-fs:~$ sudo mdadm --create /dev/md0 --level=0 --raid-device=2 /dev/sd[bc]2
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
vagrant@vm-fs:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 63.3M  1 loop  /snap/core20/1778
loop1                       7:1    0 63.3M  1 loop  /snap/core20/1822
loop2                       7:2    0 91.9M  1 loop  /snap/lxd/24061
loop3                       7:3    0 67.2M  1 loop  /snap/lxd/21835
loop4                       7:4    0 49.8M  1 loop  /snap/snapd/17950
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0  1.5G  0 part  /boot
└─sda3                      8:3    0 62.5G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
```

***
## Создайте 2 независимых PV на получившихся md-устройствах.
###
```bash
vagrant@vm-fs:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 63.3M  1 loop  /snap/core20/1778
loop1                       7:1    0 63.3M  1 loop  /snap/core20/1822
loop2                       7:2    0 49.8M  1 loop  /snap/snapd/17950
loop3                       7:3    0 91.9M  1 loop  /snap/lxd/24061
loop4                       7:4    0 67.2M  1 loop  /snap/lxd/21835
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0  1.5G  0 part  /boot
└─sda3                      8:3    0 62.5G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md127                   9:127  0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
  └─md126                   9:126  0 1018M  0 raid0 
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md127                   9:127  0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
  └─md126                   9:126  0 1018M  0 raid0 
vagrant@vm-fs:~$ sudo pvcreate /dev/md126 /dev/md127
  Physical volume "/dev/md126" successfully created.
  Physical volume "/dev/md127" successfully created.
vagrant@vm-fs:~$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               <62.50 GiB / not usable 0   
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              15999
  Free PE               8000
  Allocated PE          7999
  PV UUID               x7S6t2-at3n-E9kU-cz28-gAH3-QU9H-vyVuNf
   
  "/dev/md126" is a new physical volume of "1018.00 MiB"
  --- NEW Physical volume ---
  PV Name               /dev/md126
  VG Name               
  PV Size               1018.00 MiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               R20pzy-DoyU-i4Hs-lidF-iAzW-01DH-0EqFvt
   
  "/dev/md127" is a new physical volume of "<2.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/md127
  VG Name               
  PV Size               <2.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               DISN2X-TO41-93TD-6zM3-Z7qT-Xhdr-1PmLZS
   
vagrant@vm-fs:~$ sudo pvs
  PV         VG        Fmt  Attr PSize    PFree   
  /dev/md126           lvm2 ---  1018.00m 1018.00m
  /dev/md127           lvm2 ---    <2.00g   <2.00g
  /dev/sda3  ubuntu-vg lvm2 a--   <62.50g   31.25g
```

***
## Создайте общую volume-group на этих двух PV.
###
```bash
vagrant@vm-fs:~$ sudo vgcreate VG_0 /dev/md126 /dev/md127
  Volume group "VG_0" successfully created
vagrant@vm-fs:~$ sudo pvs
  PV         VG        Fmt  Attr PSize    PFree   
  /dev/md126 VG_0      lvm2 a--  1016.00m 1016.00m
  /dev/md127 VG_0      lvm2 a--    <2.00g   <2.00g
  /dev/sda3  ubuntu-vg lvm2 a--   <62.50g   31.25g
```

***
## Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
###
```bash
vagrant@vm-fs:~$ sudo lvcreate -L 100M VG_0 /dev/md126
  Logical volume "lvol0" created.
vagrant@vm-fs:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 63.3M  1 loop  /snap/core20/1778
loop1                       7:1    0 63.3M  1 loop  /snap/core20/1822
loop2                       7:2    0 49.8M  1 loop  /snap/snapd/17950
loop3                       7:3    0 91.9M  1 loop  /snap/lxd/24061
loop4                       7:4    0 67.2M  1 loop  /snap/lxd/21835
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0  1.5G  0 part  /boot
└─sda3                      8:3    0 62.5G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md127                   9:127  0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
  └─md126                   9:126  0 1018M  0 raid0 
    └─VG_0-lvol0          253:1    0  100M  0 lvm   
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md127                   9:127  0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
  └─md126                   9:126  0 1018M  0 raid0 
    └─VG_0-lvol0          253:1    0  100M  0 lvm  
```

***
## Создайте `mkfs.ext4` ФС на получившемся LV.
###
```bash
vagrant@vm-fs:~$ sudo mkfs.ext4 /dev/VG_0/lvol0 
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

vagrant@vm-fs:~$ lsblk -f
NAME FSTYPE LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINT
loop0
     squash                                                    0   100% /snap/core
loop1
     squash                                                    0   100% /snap/core
loop2
     squash                                                    0   100% /snap/snap
loop3
     squash                                                    0   100% /snap/lxd/
loop4
     squash                                                    0   100% /snap/lxd/
sda                                                                     
├─sda1
│                                                                       
├─sda2
│    ext4         1347b25b-64dd-4d97-80ce-90cd82397358      1.3G     7% /boot
└─sda3
     LVM2_m       x7S6t2-at3n-E9kU-cz28-gAH3-QU9H-vyVuNf                
  └─ubuntu--vg-ubuntu--lv
     ext4         d940a45b-2440-4ece-9c0c-45ced4c52e39     25.2G    13% /
sdb                                                                     
├─sdb1
│    linux_ vm-fs:1
│                 2a98afc7-86eb-566d-7264-4c4a54f33bcb                  
│ └─md127
│    LVM2_m       DISN2X-TO41-93TD-6zM3-Z7qT-Xhdr-1PmLZS                
└─sdb2
     linux_ vm-fs:0
                  53171dab-3cb0-1f55-e355-1c9ede1c3b2c                  
  └─md126
     LVM2_m       R20pzy-DoyU-i4Hs-lidF-iAzW-01DH-0EqFvt                
    └─VG_0-lvol0
       ext4         f6b93b2a-9cb3-4284-8374-1240535c3c9c                  
sdc                                                                     
├─sdc1
│    linux_ vm-fs:1
│                 2a98afc7-86eb-566d-7264-4c4a54f33bcb                  
│ └─md127
│    LVM2_m       DISN2X-TO41-93TD-6zM3-Z7qT-Xhdr-1PmLZS                
└─sdc2
     linux_ vm-fs:0
                  53171dab-3cb0-1f55-e355-1c9ede1c3b2c                  
  └─md126
     LVM2_m       R20pzy-DoyU-i4Hs-lidF-iAzW-01DH-0EqFvt                
    └─VG_0-lvol0
       ext4         f6b93b2a-9cb3-4284-8374-1240535c3c9c 
```

***
## Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.
###
```bash
vagrant@vm-fs:~$ sudo mkdir /mnt/lvo10
vagrant@vm-fs:~$ sudo mount /dev/VG_0/lvol0 /mnt/lvo10/
vagrant@vm-fs:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
udev                               948M     0  948M   0% /dev
tmpfs                              199M 1008K  198M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   31G  3.9G   26G  14% /
tmpfs                              992M     0  992M   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              992M     0  992M   0% /sys/fs/cgroup
/dev/loop0                          64M   64M     0 100% /snap/core20/1778
/dev/loop1                          64M   64M     0 100% /snap/core20/1822
/dev/loop2                          50M   50M     0 100% /snap/snapd/17950
/dev/loop3                          92M   92M     0 100% /snap/lxd/24061
/dev/loop4                          68M   68M     0 100% /snap/lxd/21835
/dev/sda2                          1.5G  110M  1.3G   8% /boot
vagrant                             78G   31G   47G  40% /vagrant
tmpfs                              199M     0  199M   0% /run/user/1000
/dev/mapper/VG_0-lvol0              93M   72K   86M   1% /mnt/lvo10
```

***
## Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
###
```bash
vagrant@vm-fs:~$ sudo wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /mnt/lvo10/test.gz
--2023-02-08 06:32:08--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 24686971 (24M) [application/octet-stream]
Saving to: ‘/mnt/lvo10/test.gz’

/mnt/lvo10/test.gz  100%[===================>]  23.54M  4.99MB/s    in 4.7s    

2023-02-08 06:32:13 (4.99 MB/s) - ‘/mnt/lvo10/test.gz’ saved [24686971/24686971]
```

***
## Прикрепите вывод `lsblk`.
###
```bash
vagrant@vm-fs:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 63.3M  1 loop  /snap/core20/1778
loop1                       7:1    0 63.3M  1 loop  /snap/core20/1822
loop2                       7:2    0 49.8M  1 loop  /snap/snapd/17950
loop3                       7:3    0 91.9M  1 loop  /snap/lxd/24061
loop4                       7:4    0 67.2M  1 loop  /snap/lxd/21835
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0  1.5G  0 part  /boot
└─sda3                      8:3    0 62.5G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md127                   9:127  0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
  └─md126                   9:126  0 1018M  0 raid0 
    └─VG_0-lvol0          253:1    0  100M  0 lvm   /mnt/lvo10
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md127                   9:127  0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
  └─md126                   9:126  0 1018M  0 raid0 
    └─VG_0-lvol0          253:1    0  100M  0 lvm   /mnt/lvo10
```

***
## Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
###
```bash
vagrant@vm-fs:~$ gzip -t /mnt/lvo10/test.gz
vagrant@vm-fs:~$ echo $?
0
```

***
## Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
###
```bash
vagrant@vm-fs:~$ sudo pvmove /dev/md126 /dev/md127
  /dev/md126: Moved: 16.00%
  /dev/md126: Moved: 100.00%
vagrant@vm-fs:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 63.3M  1 loop  /snap/core20/1778
loop1                       7:1    0 63.3M  1 loop  /snap/core20/1822
loop2                       7:2    0 49.8M  1 loop  /snap/snapd/17950
loop3                       7:3    0 91.9M  1 loop  /snap/lxd/24061
loop4                       7:4    0 67.2M  1 loop  /snap/lxd/21835
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0  1.5G  0 part  /boot
└─sda3                      8:3    0 62.5G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md127                   9:127  0    2G  0 raid1 
│   └─VG_0-lvol0          253:1    0  100M  0 lvm   /mnt/lvo10
└─sdb2                      8:18   0  511M  0 part  
  └─md126                   9:126  0 1018M  0 raid0 
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md127                   9:127  0    2G  0 raid1 
│   └─VG_0-lvol0          253:1    0  100M  0 lvm   /mnt/lvo10
└─sdc2                      8:34   0  511M  0 part  
  └─md126                   9:126  0 1018M  0 raid0 
```

***
## Сделайте `--fail` на устройство в вашем RAID1 md.
###
```bash
vagrant@vm-fs:~$ sudo mdadm /dev/md127 --fail /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md127
vagrant@vm-fs:~$ cat /proc/mdstat 
Personalities : [raid1] [raid0] [linear] [multipath] [raid6] [raid5] [raid4] [raid10] 
md126 : active raid0 sdc2[1] sdb2[0]
      1042432 blocks super 1.2 512k chunks
      
md127 : active raid1 sdc1[1] sdb1[0](F)
      2094080 blocks super 1.2 [2/1] [_U]
      
unused devices: <none>
```

***
## Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.
###
```bash
vagrant@vm-fs:~$ dmesg |grep md127
[   11.762964] md/raid1:md127: active with 2 out of 2 mirrors
[   11.763612] md127: detected capacity change from 0 to 2144337920
[ 3831.359358] md/raid1:md127: Disk failure on sdb1, disabling device.
               md/raid1:md127: Operation continuing on 1 devices.
```

***
## Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
###
```bash
vagrant@vm-fs:~$ gzip -t /mnt/lvo10/test.gz
vagrant@vm-fs:~$ echo $?
0
```

***
## Погасите тестовый хост, `vagrant destroy`.
###
```bash
vagrant@vm-fs:~$ exit
logout
Connection to 127.0.0.1 closed.
dmitriy@dmitriy-lin:~/DevOps/Vagrant$ vagrant destroy 
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```