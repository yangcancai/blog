---
title: install_erlang
date: 2020-12-23 10:56:10
tags: install
categories:
- erlang
---

## centos rpm安装erlang

### 安装依赖

```shell
sudo yum install  build-essential openssl openssl-devel unixODBC unixODBC-devel  make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz
sudo yum install unixODBC  unixODBC-devel wxBase  wxGTK SDL wxGTK-gl
```

### 安装erlang

```shel
sudo rpm -i esl-erlang_22.3.4.1-1_centos_6_amd64.rpm
```