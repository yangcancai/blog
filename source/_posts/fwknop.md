---
title: fwknop 
date: 2024-11-26 09:28:00
tags: fwknop
categories:
- fwknop
---

## fwknop
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
> doc/Makefile.am:1: error: option 'info-in-builddir' not recognized, 注释 :
```shell
 -AUTOMAKE_OPTIONS    = info-in-builddir
 +#AUTOMAKE_OPTIONS    = info-in-builddir`
 ```
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
ALLOW_IP                    23.154.152.214
ACCESS                      tcp/2194
SPA_SERVER                  127.0.0.1
KEY_BASE64                  e7USwx6Ik5LU4f3s0sBA9C5vB0y/UeQpdbDAcjT5+EY=
HMAC_KEY_BASE64             pVRDi5qu6IYT34RVQn7JNXI0ETnxVldxC+kZxMcGhjK7gF7MYTRSdDDWrJHh8IfTO4NY2zNQ6sCI6DFFSr93QA==
USE_HMAC                    Y
$ cat server/access.conf
.....
#### fwknopd access.conf stanzas ###

SOURCE              ANY
OPEN_PORTS          tcp/4022,tcp/2194,tcp/443,tcp/22000,tcp/5432
KEY_BASE64                  e7USwx6Ik5LU4f3s0sBA9C5vB0y/UeQpdbDAcjT5+EY=
HMAC_KEY_BASE64             pVRDi5qu6IYT34RVQn7JNXI0ETnxVldxC+kZxMcGhjK7gF7MYTRSdDDWrJHh8IfTO4NY2zNQ6sCI6DFFSr93QA==
REQUIRE_SOURCE_ADDRESS       Y
#PCAP_INTF                   eth0;
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
# 前端启动
$ fwknopd -f -U -a server/access.conf
```

### 测试
```shell
[root@a04f87bf9fab fwknop]# fwknop -n myssh -v
SPA Field Values:
=================
   Random Value: 5470884611265449
       Username: root
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