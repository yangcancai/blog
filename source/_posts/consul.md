---
title: consul
date: 2020-12-06 21:56:10
tags: consul
categories:
- consul
---

## Consul

### 启动

Start the Consul agent in development mode.
启动：consul agent -dev （Warning: Never run Consul in -dev mode in production）
启动读取服务配置：consul agent -dev -enable-script-checks -config-dir=./consul.d
后台启动server模式参数：
consul agent \
  -server \
  -bootstrap-expect=1 \
  -node=agent-one \
  -bind=127.0.0.1 \
  -data-dir=/tmp/consul \
  -enable-script-checks=true \
  -config-dir=/etc/consul.d \
  -ui &

Security Warning: Enabling script checks in some configurations may introduce a remote execution vulnerability which is known to be targeted by malware. In production we strongly recommend -enable-local-script-checks instead.

### 优雅停止

```shell
  $ consul leave
```

#### 读取集群成员

```shell
  $ consul members
```

#### http读取集群成员

```shell
  $ curl http//127.0.0.1:8500/v1/catalog/nodes
```

#### http注册服务

```shell
$ 
curl -XPUT http://127.0.0.1:8500/v1/agent/service/register\
 -d '{"name": "mysql", "tags": ["mysql"], "port":5672}\
"check":{\
"id":"counting_check",\
"name":"check health",\
"service_id": "mysql",\
 "tcp": "localhost:5672",\
 "interval": "10s",\
  "timeout": "1s"\
 }\
}'

```

#### http移除服务

```shell
$ curl -XPUT http://127.0.0.1:8500/v1/agent/service/deregister/mysql1
```

#### http维护模式

```shell
$ curl -XPUT http://127.0.0.1:8500/v1/agent/service/maintenance/mysql1?enable=true&reason=maintence
```

#### 获取单个服务

```shell
  $ curl http://localhost:8500/v1/catalog/service/redis1
```

#### 获取单个健康服务

```shell
  $ curl 'http://localhost:8500/v1/health/service/redis1?passing'
```