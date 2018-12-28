---
title: wkhtmltopdf插件安装错误处理
description: odoo11建议版本为0.12.1,比较坑,下载新版本的话odoo11是启动不了的,还是根据官网的推荐版本下载吧
date: 2018-05-14 15:30:20
categories:
- 技术类
tags:
- odoo
- wkhtmltopdf
---
# wkhtmltopdf插件安装错误处理

### odoo11建议版本为0.12.1

#### 查看版本信息
```
wkhtmltopdf --version
```
#### 查看软件包信息
```
pkgutil --pkg-info org.wkhtmltopdf.wkhtmltox
```
#### 查看软件包位置
```
pkgutil --files org.wkhtmltopdf.wkhtmltox
```
看到
```
usr/local/share/wkhtmltox-installer/uninstall-wkhtmltox
```
这个是卸载脚本,==先看一下这个脚本,我这个看到是空的==

#### 核对要删除的内容
```
pkgutil --only-files --files org.wkhtmltopdf.wkhtmltox
pkgutil --only-dirs --files org.wkhtmltopdf.wkhtmltox
```
#### 删除
```
pkgutil --only-files --files org.wkhtmltopdf.wkhtmltox | tr '\n' '\0' | xargs -n 1 -0 sudo rm -if
pkgutil --only-dirs --files org.wkhtmltopdf.wkhtmltox | tail -r | tr '\n' '\0' | xargs -n 1 -0 sudo rmdir
```
#### 删除安装包记录
```
sudo pkgutil --forget org.wkhtmltopdf.wkhtmltox
```
### 相关文档链接
[卸载参考文档](https://superuser.com/questions/36567/how-do-i-uninstall-any-apple-pkg-package-file)  
[ODOO V11 PDF报表打印问题](http://odoo.com.cn/posts/ubuntu-17.04-odoo-v11-and-wkhtmltopdf/)  
[wkhtmltopdf 0.12.1版本下载](
https://github.com/wkhtmltopdf/wkhtmltopdf/releases/tag/0.12.1)