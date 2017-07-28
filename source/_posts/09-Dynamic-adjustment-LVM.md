---
title: 动态调整LVM大小(XFS格式)
date: 2017-07-27 21:38:43
tags: system
categories: linux
---
{% asset_img LVM.png LVM逻辑结构 %}  

{% note warning %}
**xfs是不支持动态缩减的,可以增加**
There is no way to shrink XFS file systems.
[相关链接](http://tldp.org/HOWTO/LVM-HOWTO/reducelv.html)
但是，我们可以借助于工具搞定它
{% endnote %}

<!-- more -->

### 环境
#### 分区情况
sdb是我增加一块磁盘,8G,分成4个lvm分区
{% asset_img 01-info.png %}

上面4个分区组成了一个testVg
{% asset_img 02-VG.png %}

### 创建LV
```bash
lvcreate -L +2G -n test_lv testVg
```
生成了一个2G大小的test_lv分区
{% asset_img 03-lv.png %}


### 增加分区大小(没有格式化之前)
```bash
[root@localhost ~]# lvextend -L +1G /dev/testVg/test_lv 
Size of logical volume testVg/test_lv changed from 2.00 GiB (128 extents) to 3.00 GiB (192 extents).
Logical volume testVg/test_lv successfully resized.
```
看到已经由2G增加到3G

#### 格式化LV分区
```bash
mkfs.xfs /dev/mapper/testVg-test_lv 
```
通过 `fdisk -l` 可以看到
{% asset_img 04-fdisk.png %}

#### 挂载到系统目录
我创建了一个`/test`目录,挂载上去以后显示的3.3G,可以思考为什么
{% asset_img 05-mount.png %}
{% note info %} 提醒一下,我这边一个PE大小是16M {% endnote %}


#### 格式化以后增加分区(动态)
系统提示已经增加成功,但是通过`df -TH`没有增加
{% asset_img 06-extend.png %}
`lvdisplay`显示已经增加
{% asset_img 07-lvdis.png %}


需要执行下面的命令
`xfs_growfs /dev/mapper/testVg-test_lv`
> xfs_growfs后面的参数可以是挂载点, 也可以是lv卷名

看到已经变成4.3G了
{% asset_img 08.pngng %}
> 补充一点
> 如果想将剩余空间全部分配可以使用这个命令
> `lvextend -l 100%FREE <logvol path>`

### 缩减分区
#### 工具
缩减分区我们需要利用到一个工具
`yum -y install xfsdump`
Centos 7已经默认安装了

#### 准备测试数据
{% asset_img 09.pngg %}
{% asset_img 10.png %}


#### 备份原有数据
将现有数据使用工具备份成xfsdump格式文件
`xfsdump -f /backup/to/path/backup.xfsdump /test`
具体执行效果:
{% asset_img 11.png %}

执行完以后会生成指定命名的文件
{% asset_img 12.png %}

#### 缩减分区大小
+ 执行umount卸载分区
`umount /test`
+ 缩减分区
`lvresize -L -1.5G /dev/testVg/test_lv `
我们已经备份过数据了,确认缩减1.5G
{% asset_img 13.png %}

+ 挂载分区
`mount /dev/mapper/testVg-test_lv /test/ `
这时候是不能挂载的,因为缩减以后,要格式化分区,我们先执行格式化

+ 格式化分区(-f 强制格式化)
`mkfs.xfs -f /dev/mapper/testVg-test_lv`
然后回到上一步骤,挂载

已经挂载,但是会发现数据没了,下面就是要恢复
{% asset_img 14.png %}

#### 恢复数据
使用工具
`xfsrestore -f backup.xfsdump /test/`
看到结果成功
{% asset_img 15.png %}
去目录看看文件
{% asset_img 16.png %}

**空间缩减了,数据恢复成功**