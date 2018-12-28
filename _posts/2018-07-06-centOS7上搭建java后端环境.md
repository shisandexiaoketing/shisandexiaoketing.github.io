---
title: centOS7上搭建java后端环境
description: 刚装好的centOs7 安装docker,安装docker仓库(因为在搭建k8s时遇到pull不下项目的问题,所以准备这么解决),jdk8,jenkins和配置等
date: 2018-07-06 22:02:12
categories:
- 技术类
tags:
- linux
- centOs7
- docker
---

# centOS7 java服务器环境搭建

#### 安装docker和本地镜像仓库
##### 安装docker
```
yum -y install docker
```
##### 启动
```
systemctl start docker
```
##### 搭建私有仓库
```
docker pull registry
```
##### 防火墙添加运行5000端口
```
iptables -I INPUT 1 -p tcp --dport 5000 -j ACCEPT
```
##### 下载完之后我们通过该镜像启动一个容器
>参数说明： 
-v /opt/registry:/tmp/registry :默认情况下，会将仓库存放于容器内的/tmp/registry目录下，指定本地目录挂载到容器 
–privileged=true ：CentOS7中的安全模块selinux把权限禁掉了，参数给容器加特权，不加上传镜像会报权限错误(OSError: [Errno 13] Permission denied: ‘/tmp/registry/repositories/liibrary’)或者（Received unexpected HTTP status: 500 Internal Server Error）错误

##### 客户端上传镜像
>修改/etc/sysconfig/docker（Ubuntu下配置文件地址为：/etc/init/docker.conf），增加启动选项(已有参数的在后面追加)，之后重启docker，不添加报错，https证书问题。

```
OPTIONS='--insecure-registry 192.168.16.12:5000'    #CentOS 7系统
other_args='--insecure-registry 192.168.16.12:5000' #CentOS 6系统
```

>因为Docker从1.3.X之后，与docker registry交互默认使用的是https，而此处搭建的私有仓库只提供http服务 
在docker公共仓库下载一个镜像  

==这个改完重启一下docker==
```
sudo restart docker
```

##### 从公有库下载一个镜像  
```
docker pull mysql:5.6
```
##### 修改一下该镜像的tag
```
docker tag docker.io/mysql:5.6 192.168.16.12:5000/mysql:5.6
```
##### 把打了tag的镜像上传到私有仓库
```
docker push 192.168.16.12:5000/mysql:5.6
```
##### 客户端添加私有仓库地址
```
# 添加这一行
ADD_REGISTRY='--add-registry 192.168.16.12:5000'
```
##### 删掉从远程下载的镜像,pull本地仓库的镜像
```
docker pull 192.168.16.12:5000/mysql
```
#### JDK安装
##### 更新系统软件
```
yum update
```
##### 查找系统已安装的jdk组件
```
rpm -qa | grep -E '^open[jre|jdk]|j[re|dk]'
```
##### 查看java版本
```
java -version
```
##### 卸载以前已有的jdk
```
yum remove java-1.6.0-openjdk
yum remove java-1.7.0-openjdk
```
##### 在/usr目录中先建名为java的文件夹
```
mkdir /usr/java
```
>下载jdk-8u171-linux-x64.tar.gz包，并上传至服务器/usr/local文件夹中。
解压jdk-8u111-linux-x64.tar.gz包至/usr/local/jdk1.8.0_171文件夹

```
tar -xvf jdk-8u171-linux-x64.tar.gz
```
##### 添加到环境变量
>编辑/etc/profile文件，在export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL下面添加如下代码：

```
#jdk
export JAVA_HOME=/usr/java/jdk1.8.0_171
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
##### 执行命令使配置生效
```
source /etc/profile
```
##### 验证，是否安装成功
```
java -version
```
#### JENKINS 安装
##### 安装wget
```
yum -y install wget
```
##### 下载依赖
```
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
```
##### 导入秘钥
```
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```
##### 安装
```
yum install jenkins
```
##### 查找jenkins安装路径
```
rpm -ql jenkins
```
>jenkins相关目录释义：
（1）/usr/lib/jenkins/：jenkins安装目录，war包会放在这里。
( 2 ) /etc/sysconfig/jenkins：jenkins配置文件，“端口”，“JENKINS_HOME”等都可以在这里配置。
（3）/var/lib/jenkins/：默认的JENKINS_HOME。
（4）/var/log/jenkins/jenkins.log：jenkins日志文件。

##### 查找jenkins端口
```
vim /etc/sysconfig/jenkins
：set ignorecase
/jenkins_port 回车
```
##### 查看端口占用情况
```
netstat -ntlp
```
##### 启动jenkins
```
java -jar /usr/lib/jenkins/jenkins.war --httpPort=8080
```
##### 修改端口启动
```
#端口改为8899
java -jar /usr/lib/jenkins/jenkins.war --ajp13Port=-1 --httpPort=8899
#启动
java -jar /usr/lib/jenkins/jenkins.war --httpPort=8899
```

##### 后台启动
```
nohup java -jar /usr/lib/jenkins/jenkins.war --httpPort=8080 &
```
##### 放开8080端口
```
iptables -I INPUT 1 -p tcp --dport 8080 -j ACCEPT
```
##### 权限问题进入设置页面空白页了,
>找到.jenkins/config.xml文件：
替换为： 

任何用户可以做任何事(没有任何限制)
```
<authorizationStrategy class="hudson.security.AuthorizationStrategy$Unsecured"/>
```
登录用户可以做任何事
```
<authorizationStrategy class="hudson.security.FullControlOnceLoggedInAuthorizationStrategy"/>
```

##### jenkins停止
```
访问
http://192.168.16.12:8080/exit
```
##### 配置安装docker和maven插件
>jenkins不能使用maven3.2以上的版本,下载了maven最新版jenkins一直找不到,索性用jenkins安装3.0的版本,ok了

#### docker compose 安装
##### docker-compose是部署多个容器的重要工具。 

* 首先检查linux有没有安装python-pip包，直接执行 
```
yum install python-pip -y
```
* 没有python-pip包就执行命令
```
yum -y install epel-release
```

* 执行成功之后，再次执行
```
yum install python-pip
```
* 对安装好的pip进行升级
```
pip install --upgrade pip
```
* 至此，pip工具就安装好了，下面使用pip  
* 安装docker-compose
```
pip install docker-compose
```
输入```docker-compose```验证，报错的话就百度，下面这个报错：
>[root@localhost ~]# docker-compose  
Traceback (most recent call last):  
  File "/usr/bin/docker-compose", line 5, in <module>  
    from pkg_resources import load_entry_point  
  File "/usr/lib/python2.7/site-packages/pkg_resources.py", line 3011, in <module>  
    parse_requirements(__requires__), Environment()  
  File "/usr/lib/python2.7/site-packages/pkg_resources.py", line 626, in resolve  
    raise DistributionNotFound(req)  
pkg_resources.DistributionNotFound: backports.ssl-match-hostname>=3.5 

解决：
```
pip install --upgrade backports.ssl_match_hostname
```

### windows下编写的Shell脚本在Linux下运行错误的解决方法

>当在Linux下写好一个脚本之后保存在windows上，在Windows上修改以后再传到Linux上，可能脚本就不能执行了。
出现这种错误的原因是因为：CR/LF问题，在dos/window下按一次回车键实际上输入的是“回车（CR)”和“换行（LF）”，而Linux/unix下按一次回车键只输入“换行（LF）”，所以修改的sh文件在每行都会多了一个CR，所以Linux下运行时就会报错找不到命令。

##### 举出两种解决方法：
* 在editplus中“文档->文件格式(CR/LF)->UNIX”，这样Linux下就能按unix的格式保存文件 
* 在vim中，输入:set ff=unix，同样也是转换成unix的格式。

#### 一些问题
切换到新的服务器的mongo地址之后,本地外部访问是ok的,服务器端构建服务启动报错"No route to host"

解决办法:将27017端口放开
```
iptables -I INPUT 1 -p tcp --dport 27017 -j ACCEPT

```
同理其他问题也是放开相关端口

##### jenkins忘记密码了怎么办-jenkins找回密码

```
cd /root/.jenkins/users/admin
```
打开config.xml
```
<?xml version='1.0' encoding='UTF-8'?>
<user>
  <fullName>admin</fullName>
  <description>总管理员账号</description>
  <properties>
    <jenkins.security.ApiTokenProperty>
      <apiToken>rWArknUk9PnLS7riVJGISU/HFkjErmpNNNuiDC31aFd0SjdAh0ih3tN8GDkC94Nm</apiToken>
    </jenkins.security.ApiTokenProperty>
    <jenkins.security.LastGrantedAuthoritiesProperty>
      <roles>
        <string>authenticated</string>
      </roles>
      <timestamp>1475071638132</timestamp>
    </jenkins.security.LastGrantedAuthoritiesProperty>
    <hudson.model.MyViewsProperty>
      <primaryViewName></primaryViewName>
      <views>
        <hudson.model.AllView>
          <owner class="hudson.model.MyViewsProperty" reference="../../.."/>
          <name>All</name>
          <filterExecutors>false</filterExecutors>
          <filterQueue>false</filterQueue>
          <properties class="hudson.model.View$PropertyList"/>
        </hudson.model.AllView>
      </views>
    </hudson.model.MyViewsProperty>
    <hudson.model.PaneStatusProperties>
      <collapsed/>
    </hudson.model.PaneStatusProperties>
    <hudson.security.HudsonPrivateSecurityRealm_-Details>
      <passwordHash>#jbcrypt:$2a$10$NqPv3NpgxkpQi/ffEsEkhuMZYpbKc5cVVrP60cD6MX5IujYkLlOGm</passwordHash>
    </hudson.security.HudsonPrivateSecurityRealm_-Details>
    <org.jenkinsci.main.modules.cli.auth.ssh.UserPropertyImpl>
      <authorizedKeys></authorizedKeys>
    </org.jenkinsci.main.modules.cli.auth.ssh.UserPropertyImpl>
    <hudson.search.UserSearchProperty>
      <insensitiveSearch>false</insensitiveSearch>
    </hudson.search.UserSearchProperty>
  </properties>
</user>
```
>如上面的配置文件，把passwordHash改成上面的值（对应密码是123456）,账户就是这个文件夹的名称，改好后登录jenkins，在管理平台上修改密码即可。

##### 停止jenkins

```
ps -ef | grep jenkins
```
显示如下:
```
[root@localhost admin]# ps -ef | grep jenkins
root     11996 10614  8 19:11 pts/0    00:01:51 java -jar /usr/lib/jenkins/jenkins.war --httpPort=8080
root     20291 10614  0 19:34 pts/0    00:00:00 grep --color=auto jenkins

```
执行这行命令杀死进程
```
kill -s 9 11996
```
