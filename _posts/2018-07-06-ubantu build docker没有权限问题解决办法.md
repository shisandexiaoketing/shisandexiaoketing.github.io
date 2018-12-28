---
title: ubantu build docker没有权限问题解决办法
description: ubantu build docker没有权限问题解决办法
date: 2018-07-06 21:53:20
categories:
- 技术类
tags:
- linux
- ubantu
- docker
---

# ubantu build docker没有权限问题解决办法

报错信息
```
 org.apache.http.impl.execchain.RetryExec execute
INFO: Retrying request to {}->unix://localhost:80
```
解决办法
```
chmod 777 /var/run/docker.sock
或者
usermod -a -G docker jenkins
```
**这里用的第一种方法已经解决**

###### Docker daemon日志的位置

* Ubuntu - /var/log/upstart/docker.log
* Boot2Docker - /var/log/docker.log
* Debian GNU/Linux - /var/log/daemon.log
* CentOS - /var/log/messages | grep docker
* Fedora - journalctl -u docker.service
* Red Hat Enterprise Linux Server - /var/log/messages | grep docker


