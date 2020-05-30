---
title: 使用alternatives管理java环境
date: 2020-03-1 19:36:26
tags: linux
categories: system
---

{% note info %}

Linux自带的java环境都比较老，目前的ES，tarBall都含有最新的JDK，不需要去JAVA官网下载新的，直接使用自带的神器alternatives安装即可管理使用

{% endnote %}

<!-- more -->

### 下载安装包
通过官网下载
```Bash
[root@log elasticsearch-7.6.1]# cd /opt/elastic
[root@log elasticsearch-7.6.1]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.2-linux-x86_64.tar.gz
```

解压以后能看到有jdk包
```bash
[root@log elasticsearch-7.6.1]# tar elasticsearch-7.6.2-linux-x86_64.tar.gz
[root@log elasticsearch-7.6.1]# ll
total 564
drwxr-xr-x  2 elasticsearch elasticsearch   4096 Feb 29 08:18 bin
drwxr-xr-x  2 elasticsearch elasticsearch    178 Apr 27 10:21 config
drwxr-xr-x  3 elasticsearch elasticsearch     19 Mar 27 14:38 data
drwxr-xr-x  9 elasticsearch elasticsearch    107 Feb 29 08:18 jdk
drwxr-xr-x  3 elasticsearch elasticsearch   4096 Feb 29 08:18 lib
-rw-r--r--  1 elasticsearch elasticsearch  13675 Feb 29 08:13 LICENSE.txt
drwxr-xr-x  2 elasticsearch elasticsearch   4096 May 12 11:04 logs
drwxr-xr-x 38 elasticsearch elasticsearch   4096 Feb 29 08:18 modules
-rw-r--r--  1 elasticsearch elasticsearch 523209 Feb 29 08:18 NOTICE.txt
-rw-r--r--  1 elasticsearch elasticsearch      5 May 12 11:05 pid
drwxr-xr-x  2 elasticsearch elasticsearch      6 Feb 29 08:18 plugins
-rw-r--r--  1 elasticsearch elasticsearch   8164 Feb 29 08:13 README.asciidoc

```

### alternatives管理java

1. 目前只有1个java环境
```bash
[root@log ~]# alternatives --config java

There are 1 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
*+ 1           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64/jre/bin/java)
```

2. 添加es的jdk环境
```bash
# 添加java 和 javac
[root@log ~]# alternatives --install /usr/bin/java java /opt/elasticsearch/elasticsearch-7.6.1/jdk/bin/java 100
[root@log ~]# alternatives --install /usr/bin/javac javac /opt/elasticsearch/elasticsearch-7.6.1/jdk/bin/javac 100
```
> 参数说明：
> `/usr/bin/java` 为`ln -s`的目标路径
> `java` 为`config`名
> `/opt/elasticsearch/elasticsearch-7.6.1/jdk/bin/java`为java所在的目录
> `100`为优先级

3. 选择默认的Java版本
```bash
# 选择相应的数字，选中的环境前面会有个+号
[root@log ~]# alternatives --config java

There are 2 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
*  1           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64/jre/bin/java)
 + 2           /opt/elasticsearch/elasticsearch-7.6.1/jdk/bin/java

Enter to keep the current selection[+], or type selection number: 2

# javac也进行同样的操作
[root@log ~]# alternatives --config javac

There are 2 programs which provide 'javac'.

  Selection    Command
-----------------------------------------------
*  1           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64/bin/javac)
 + 2           /opt/elasticsearch/elasticsearch-7.6.1/jdk/bin/javac

Enter to keep the current selection[+], or type selection number: 2
```

4. 验证是否正确
```bash
[root@log ~]# java -version
openjdk version "13.0.2" 2020-01-14
OpenJDK Runtime Environment AdoptOpenJDK (build 13.0.2+8)
OpenJDK 64-Bit Server VM AdoptOpenJDK (build 13.0.2+8, mixed mode, sharing)
```