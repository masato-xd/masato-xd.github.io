---
title: Centos7配置代理
date: 2018-03-10 13:34:08
tags: system
categories: linux
---


{% note info %}

Privoxy 只是一个HTTP代理。
Privoxy 自己并【没有】对应的“代理服务器”。
它仅仅是一款“客户端的代理工具”。
你自己需要有服务器

{% endnote %}

<!-- more -->

### 安装shadowsocks

`pip install shadowsocks`

**配置 vim /etc/shadowsocks**
```bash
{
  "server":"6.2.5.22",     # 你的那个服务器 
  "server_port":9528,            # 提供服务的端口
  "local_address": "127.0.0.1",  
  "local_port":1080,             
  "password":"test@123.com",   # 密码   
  "timeout":300,                 
  "method":"aes-256-cfb",        
  "workers": 1                   
}
```

**执行启动shadowsocks**

```bash
nohup sslocal -c /etc/shadowsocks.json  &
```

**测试**

运行 `curl --socks5 127.0.0.1:1985 http://httpbin.org/ip`如果返回你的 ss 服务器 ip 则测试成功：
```bash
{
  "origin": "x.x.x.x"       #你的 ss 服务器 ip
}
```

### 下面安装Privoxy
`Privoxy` 强烈不建议使用`root`用户运行，所以我们使用`useradd privoxy`新建一个用户.

**进入privoxy目录**
```bash
autoheader && autoconf
./configure
make && make install
```

**修改配置**
```bash
vim /usr/local/etc/privoxy/config

# 找到以下两句，确保没有注释掉
listen-address 127.0.0.1:8118   # 8118 是默认端口，不用改，下面会用到  783行
forward-socks5t / 127.0.0.1:1080 . # 这里的端口写 shadowsocks 的本地端口（注意最后那个 . 不要漏了）  1336行
```

**启动**
```bash
privoxy --user privoxy /usr/local/etc/privoxy/config
```

### 配置启动环境变量
```bash
vim /etc/profile
export http_proxy=http://127.0.0.1:8118    #这里的端口和上面 privoxy 中的保持一致
export https_proxy=http://127.0.0.1:8118

source /etc/profile

curl www.google.com
```

### 电脑重启执行(可以写入到`rc.local`中)
```bash
nohup sslocal -c /etc/shadowsocks.json &
privoxy --user privoxy /usr/local/etc/privoxy/config
```
