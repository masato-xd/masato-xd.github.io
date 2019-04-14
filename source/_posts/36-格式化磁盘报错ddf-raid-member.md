---
title: 格式化磁盘报错ddf_raid_member
date: 2019-04-14 19:20:39
tags: system
categories: linux
---

{% note info %}

将一块带有 raid 信息的磁盘接入新的服务器,进行格式化的时候,会提示`ddf_raid_member`报错信息
清除即可

{% endnote %}

<!-- more -->


系统插入了一块带有RAID信息的磁盘，格式化的时候报错

```shell
root@sjz-db1:~# mkfs.ext4 /dev/sdb 
mke2fs 1.42.13 (17-May-2015)
/dev/sdb contains a ddf_raid_member file system
Proceed anyway? (y,n) y
/dev/sdb is apparently in use by the system; will not make a filesystem here!
```

解决办法是清除RAID信息
```shell
root@sjz-db1:~# cat /proc/mdstat 
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md126 : inactive sdb[0]
      468320256 blocks super external:/md127/0
       
md127 : inactive sdb[0](S)
      531288 blocks super external:ddf
       
unused devices: <none>
root@sjz-db1:~# mdadm -S /dev/md126      
mdadm: stopped /dev/md126
root@sjz-db1:~# mdadm -S /dev/md127
mdadm: stopped /dev/md127

root@sjz-db1:~# mkfs.ext4 /dev/sdb          

mke2fs 1.42.13 (17-May-2015)
/dev/sdb contains a ddf_raid_member file system
Proceed anyway? (y,n) y
Discarding device blocks: done                            
Creating filesystem with 117212886 4k blocks and 29310976 inodes
Filesystem UUID: c57fb9ab-d40a-4955-a4cf-69cf86d2c69b
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
        102400000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done     

```