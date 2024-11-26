---
title: fwknop 
date: 2024-11-26 09:28:00
tags: fwknop
categories:
- fwknop
---

## fwknop-2.6.11
### 安装
```shell
git clone https://github.com/mrash/fwknop.git
cd fwknop/
yum install -y libpcap libpcap-devel autoconf automake libtool texinfo which
autoreconf -vi
automake --version
./autogen.sh
./configure
make
make install
```
### 问题
> 1. doc/Makefile.am:1: error: option 'info-in-builddir' not recognized, 注释 :
```shell
 -AUTOMAKE_OPTIONS    = info-in-builddir
 +#AUTOMAKE_OPTIONS    = info-in-builddir`
 ```
### 外网无法访问62201
1. 使用tcpdump验证数据包到达目标机器： `tcpdump -i eth0 udp port 62201`
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
07:00:30.340096 IP x.x.x.x.54914 > C20240909202534.62201: UDP, length 225
2. 防火墙记录丢弃的数据包: `iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: " --log-level 4`
3. 查看系统日志发现数据包被丢弃了: `tail -f /var/log/messages`
  Nov 26 06:52:26 C20240909202534 kernel: IPTables-Dropped: IN=eth0 OUT= MAC=12:bf:fc:00:02:53:ac:75:1d:80:2c:94:08:00 SRC=x.x.x.x DST=x.x.x.x LEN=253 TOS=0x00 PREC=0x00 TTL=53 ID=26683 PROTO=UDP SPT=57822 DPT=62201 LEN=233
4. 查看log的编号: `iptables -nvL --line-numbers`  
5. 在log的前面开放62201: `iptables -I INPUT 8 -p udp --dport 62201 -j ACCEPT`

### 在docker中测试
```shell
$ docker run --privileged -it centos:7 /bin/bash
# 替换yum源
[root@7a0b6d82e21a]# curl -O https://file.tsyvps.com/yumcentos7.sh && chmod +x yumcentos7.sh && ./yumcentos7.sh
```

### 配置
```shell
# 生成配置到~/.fwknoprc
$ fwknop -A tcp/22 -a 192.168.163.1 -D myssh -p 62201 -P udp --key-gen --use-hmac --save-rc-stanza
$ cat ~/.fwknoprc
[default]

[myssh]
SPA_SERVER_PROTO            udp
SPA_SERVER_PORT             62201
ALLOW_IP                    192.168.163.1
ACCESS                      tcp/2194
# 修改这个ip
SPA_SERVER                  myssh
KEY_BASE64                  e7USwx6Ik5LU4f3s0sBA9C5vB0y/UeQpdbDAcjT5+EY=
HMAC_KEY_BASE64             pVRDi5qu6IYT34RVQn7JNXI0ETnxVldxC+kZxMcGhjK7gF7MYTRSdDDWrJHh8IfTO4NY2zNQ6sCI6DFFSr93QA==
USE_HMAC                    Y
# 增加用户名
SPOOF_USER                  who
$ cat server/access.conf
.....
#### fwknopd access.conf stanzas ###

SOURCE              ANY
OPEN_PORTS          tcp/4022,tcp/2194,tcp/443,tcp/22000,tcp/5432
KEY_BASE64                  e7USwx6Ik5LU4f3s0sBA9C5vB0y/UeQpdbDAcjT5+EY=
HMAC_KEY_BASE64             pVRDi5qu6IYT34RVQn7JNXI0ETnxVldxC+kZxMcGhjK7gF7MYTRSdDDWrJHh8IfTO4NY2zNQ6sCI6DFFSr93QA==
REQUIRE_SOURCE_ADDRESS       Y
REQUIRE_USERNAME             who
# If you want to use GnuPG keys then define the following variables
#
#GPG_HOME_DIR           /homedir/path/.gnupg
#GPG_DECRYPT_ID         ABCD1234
#GPG_DECRYPT_PW         __CHANGEME__

# If you want to require GPG signatures:
#GPG_REQUIRE_SIG                    Y
#GPG_IGNORE_SIG_VERIFY_ERROR        N
#GPG_REMOTE_ID                      1234ABCD
$ 
```

### 启动
```shell
# 后端启动
$ fwknopd -U -a server/access.conf
# 关闭
$ fwknopd -K
# 前端启动
$ [root@C20240909202534 fwknop]# fwknopd -f -U -a server/access.conf
Warning: REQUIRE_SOURCE_ADDRESS not enabled for access stanza source: 'ANY'
Starting fwknopd
Added jump rule from chain: INPUT to chain: FWKNOP_INPUT
iptables 'comment' match is available
Kicking off UDP server to listen on port 62201.


(stanza #1) SPA Packet from IP: x.x.x.x received with access source match

```

### 测试
```shell
# 服务器抓包62201
$ tcpdump -i eth0 udp port 62201
# 敲门
$ fwknop -n myssh -v
SPA Field Values:
=================
   Random Value: 5470884611265449
       Username: who
      Timestamp: 1732601087
    FKO Version: 3.0.0
   Message Type: 1 (Access msg)
 Message String: 192.168.163.1,tcp/22
     Nat Access: <NULL>
    Server Auth: <NULL>
 Client Timeout: 0
    Digest Type: 3 (SHA256)
      HMAC Type: 3 (SHA256)
Encryption Type: 1 (Rijndael)
Encryption Mode: 2 (CBC)
   Encoded Data: 5470884611265449:cm9vdA:1732601087:3.0.0:1:MTkyLjE2OC4xNjMuMSx0Y3AvMjI
SPA Data Digest: jOsmgIhZf2wn5sQPtnQxZ/d5uRPJZk07T567rM29pMc
           HMAC: 0zKRN7h4GFdJX7TUcoR5EbWizRljFTlESqtIDsGiJ3A
 Final SPA Data: 9WhOfa1TU3xRw8nJaeoSGrB1SHQzLami92aUmTH4id3H33gVEHz5XHwBzK8rNKQC0TdOdxZQ4xIkFuLjLBNWshRGAHxBrcJB5u61o3PcNJczCYa1OLMhRKsAEvU/EXZsyWYYsC+I1H3KwMxiARTNcZnPhOtYU/bBGKa6+bcp3KTcJDujSD8LhM0zKRN7h4GFdJX7TUcoR5EbWizRljFTlESqtIDsGiJ3A

Generating SPA packet:
            protocol: udp
         source port: <OS assigned>
    destination port: 62201
             IP/host: 127.0.0.1
send_spa_packet: bytes sent: 225
```