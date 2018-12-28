---
title: ubantu上搭建java后端环境
description: ubantu 环境搭建,mysql,redismongodb,rabbitMq,jenkins等,docker预装
date: 2018-07-06 21:59:20
categories:
- 技术类
tags:
- linux
- ubantu
- docker
---

# ubantu java服务器环境搭建

##### mysql 
```
docker pull mysql:5.6
docker run -p 3306:3306 --name mymysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
```

##### redis
```
docker pull  redis:3.2 
docker run -p 6379:6379 -v $PWD/data:/data  -d redis:3.2 redis-server --appendonly yes
```

##### mongodb
```
docker run -p 27017:27017 --name mongo -v /root/docker/data/mongo:/data/db -d mongo:latest 
```

##### RabbitMQ
```
docker run -d --hostname rabbitmq --name rabbitmq  -p 5672:5672 -p 7080:15672 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=123456 rabbitmq:3-management
```

##### jenkins
```
wget -q -O - https://jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
```
```
sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```
```
sudo apt-get update

sudo apt-get install jenkins
```
```
cat /var/lib/jenkins/secrets/initialAdminPassword
```
复制初始密码  

服务的启动/停止：
```
sudo service jenkins start  

sudo service jenkins stop  
```
升级
```
sudo apt-get update  

sudo apt-get install jenkins
```
路径：

访问路径：http://localhost:8080  

安装路径：/var/lib/jenkins  

日志路径：/var/log/jenkins

创建的项目目录当然在jenkins安装目录下的jobs里面。  
==用户名密码:==

```
admin
123456
```