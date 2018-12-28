---
title: PostgreSQL使用
description: odoo系列使用的数据库--PostgreSQL
date: 2018-05-14 15:32:20
categories:
- 技术类
tags:
- odoo
- PostgreSQL
---
# PostgreSQL使用
### 安装
```
brew install postgresql
```
### 查看安装版本
```
pg_ctl -V
```
### 初始化
```
initdb /usr/local/var/postgres
```
==这里我初始化失败，我也不知道为什么，然后我就直接启动服务就启动成功了！所以我认为这里不需要初始化。==

### 启动服务:
```
pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
```
### 停止服务：
```
pg_ctl -D /usr/local/var/postgres stop -s -m fast
```
### 查看服务状态：
```
pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log status
```
### 设置开机启动（可选）：
```
ln -sfv /usr/local/opt/postgresql/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
```
### 卸载postgreSQL
```
brew uninstall postgresql
```
### 取消开机启动服务（如果配置了）：
```
launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
rm -rf ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
```

## 创建数据库和账户
### mac安装PostgreSQL后不会创建用户名数据库，执行命令：
```
createdb
```
### 然后登录PostgreSQL控制台：
```
psql
```
==psql连接数据库默认选用的是当前的系统用户==  
使用```\l```命令列出所有的数据库，看到已存在用户同名数据库、postgres数据库，但是postgres数据库的所有者是当前用户，没有postgres用户。按```:q```退出

## 接下来操作下面命令(1-5)
### 1.创建postgres用户
```
CREATE USER postgres WITH PASSWORD 'XXXXXX';
```
==密码自己替换==
### 2.删除默认生成的postgres数据库
```
DROP DATABASE postgres;
```
### 3.创建属于postgres用户的postgres数据库
```
CREATE DATABASE postgres OWNER postgres;
```
### 4.将数据库所有权限赋予postgres用户
```
GRANT ALL PRIVILEGES ON DATABASE postgres to postgres;
```
### 5.给postgres用户添加创建数据库的属性
```
ALTER ROLE postgres CREATEDB;
```
==这样就可以使用postgres作为数据库的登录用户了，并可以使用该用户管理数据库==
## 登陆控制台指令
```
psql -U [user] -d [database] -h [host] -p [port]
```
==-U指定用户，-d指定数据库，-h指定服务器，-p指定端口==

### 完整的登录命令，比如使用postgres用户登录
```
psql -U postgres -d postgres
```
### 之前我们直接使用```psql```登录控制台，实际上使用的是默认数据
```
user：当前mac用户
database：用户同名数据库
主机：localhost
端口号：5432，postgresql的默认端口是5432
```
### 常用控制台指令
```
\password：设置当前登录用户的密码
\h：查看SQL命令的解释，比如\h select。
\?：查看psql命令列表。
\l：列出所有数据库。
\c [database_name]：连接其他数据库。
\d：列出当前数据库的所有表格。
\d [table_name]：列出某一张表格的结构。
\du：列出所有用户。
\e：打开文本编辑器。
\conninfo：列出当前数据库和连接的信息。
\password [user]: 修改用户密码
\q：退出
```
## 使用PostgreSQL
以下命令都在```postgres=#``` 环境下

### 修改用户密码
```
ALTER USER postgres WITH PASSWORD 'XXXXXX'
```
==密码自己替换==  
出现ALTER ROLE, 代表修改角色成功

##### 创建和删除数据库用户
##### 创建user1用户：
```
CREATE USER user1 WITH PASSWORD 'XXXX'
```
##### 查看数据库用户列表：```\du```

##### 删除数据库用户：```rop user user1;```

### 创建和删除数据库
##### 创建数据库：```create database testdb;```

##### 查看数据库列表：```\l```

##### 删除数据库：```drop database db1;```

### 创建和删除数据表
##### 选择数据库：```\c DatabaseName```，比如```\c testdb```

##### 创建数据库表：
```
CREATE TABLE COMPANY( ID INT PRIMARY KEY NOT NULL, NAME TEXT NOT NULL, AGE INT NOT NULL, ADDRESS CHAR(50), SALARY REAL);
```

##### 删除数据库表： drop table company;

##### 查看数据库信息：\d

##### 查询数据：select * from company
### 下载和使用客户端管理工具
[pgAdmin4](https://www.pgadmin.org/download/)

