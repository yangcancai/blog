---
title: 代理协议
date: 2022-03-09 16:57:00
categories:
- 代理
tags: 
- http
- https
---

  http和https代理协议分析，我们使用的手机wifi手动设置代理和电脑浏览器设置的代理基本上是使用http、https和socks。
今天主要通过wireshark抓包来分析http和https代理协议

## 抓包分析
http直接访问
```text
    GET / HTTP/1.1\r\n
    Host: 192.168.3.14\r\n
    Connection: keep-alive\r\n
    Upgrade-Insecure-Requests: 1\r\n
    User-Agent: Mozilla/5.0 (Linux; Android 10; HarmonyOS; AQM-AL00; HMSCore 6.4.0.302) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.105 HuaweiBrowser/12.0.4.300 Mobile Safari/537.36\r\n
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9\r\n
    Accept-Encoding: gzip, deflate\r\n
    Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
```
http代理访问
```text
    GET http://192.168.3.14/ HTTP/1.1\r\n
    Host: 192.168.3.14\r\n
    Proxy-Connection: keep-alive\r\n
    Proxy-Authorization: Basic encoded-credentials\r\n
    Upgrade-Insecure-Requests: 1\r\n
    User-Agent: Mozilla/5.0 (Linux; Android 10; HarmonyOS; AQM-AL00; HMSCore 6.4.0.302) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.105 HuaweiBrowser/12.0.4.300 Mobile Safari/537.36\r\n
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9\r\n
    Accept-Encoding: gzip, deflate\r\n
    Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
```

https代理访问
```text
    CONNECT 192.168.3.14:443 HTTP/1.1\r\n
    Host: 192.168.3.14:443\r\n
    Proxy-Connection: keep-alive\r\n
    Proxy-Authorization: Basic encoded-credentials\r\n
    User-Agent: Mozilla/5.0 (Linux; Android 10; HarmonyOS; AQM-AL00) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.105 Mobile Safari/537.36\r\n
    \r\n
```
通过以上分析我们知道,通过代理访问的http协议path是全路径，http头Connection变为Proxy-Connection,
https协议首先会调用CONNECT方法跟代理建立连接,如果代理服务器需要授权，那么使用Proxy-Authorization

