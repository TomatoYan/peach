---
title: jvisualvm远程监控Tomcat
tags: jvm
abbrlink: 40878
date: 2017-08-09 18:02:17
---
## VisualVM：多合一故障处理工具
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VisualVM(All-in-One Java Trouble Tool)是到目前为止随JDK发布功能最强大的运行监视和故障处理程序。而且VisualVM不需要被监视的程序基于特殊Agent运行，因此它对实际性能的影响很小，使得它可以直接应用在生产环境中。
界面如下：
![界面1](http://img.blog.csdn.net/20170809183644088?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaV9hbV90b21hdG8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<!-- more -->

![界面2](http://img.blog.csdn.net/20170809183710495?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaV9hbV90b21hdG8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
## VisualVM的功能
VisualVM可以做到：

- 显示虚拟机进程以及进程的配置、环境信息(jsp、jinfo)。
- 监视应用程序的CPU、GC、堆、方法区以及线程的信息(jstat、jstack)。
- dump以及分析堆转储快照(jmap、jhat)。
- 方法级的程序运行性能分析，找出被调用最多、运行时间最长的方法。
- 离线程序快照：收集程序的运行时配置、线程dump、内存dump等信息建立一个快照，可以将快照发送开发者处进行Bug反馈。

## VisualVM安装插件
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于墙的原因，所以需要采取些措施，才能顺利的下载安装插件。
![插件](http://img.blog.csdn.net/20170810153927241?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaV9hbV90b21hdG8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## jvisualvm远程监控Tomcat
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;进入我们的正题，利用jvisualvm远程监控Tomcat服务器。这里的Tomcat部署在远程Linux服务器上。

### 1.修改Tomcat下bin目录下的startup.sh
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在startup.sh倒数第二行，也就是exec "$PRGDIR"/"$EXECUTABLE" start "$@"上一行添加如下内容：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`export CATALINA_OPTS="$CATALINA_OPTS
-Dcom.sun.management.jmxremote
-Djava.rmi.server.hostname=192.168.1.130
-Dcom.sun.management.jmxremote.port=8888
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=true
-Dcom.sun.management.jmxremote.password.file=../conf/jmxremote.password
-Dcom.sun.management.jmxremote.access.file=../conf/jmxremote.access"`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;各个参数的说明如下：

    -Dcom.sun.management.jmxremote   #启用JMX远程监控
	-Djava.rmi.server.hostname=192.168.1.130  #Tomcat服务器地址
	-Dcom.sun.management.jmxremote.port=8888  #jmx连接端口
	-Dcom.sun.management.jmxremote.ssl=false   #是否ssl加密
	-Dcom.sun.management.jmxremote.authenticate=false   #远程连接不需要密码认证
	-Dcom.sun.management.jmxremote.password.file=../conf/jmxremote.password   #指定连接的用户名和密码配置文件
	-Dcom.sun.management.jmxremote.access.file=../conf/jmxremote.access   #指定连接的用户所拥有权限的配置文件

### 2.修改服务器上Tomcat的配置：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先需要修改两个配置文件，首先进入 JAVA_HOME\jre\lib\management 目录下，拷贝和修改jmxremote.password.template为jmxremote.password，操作为：
取消最后两行代码的注释，

	\# monitorRole  QED   --->   monitorRole  QED
	\# controlRole   R&D   --->   controlRole   R&D
拷贝和修改文件jmxremote.access，操作为：

	monitorRole   readonly
	controlRole   readwrite

### 3.将文件拷贝至服务器上Tomcat下的conf目录下
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;即和tomcat的配置文件放在一起

### 4.修改两个文件权限
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;修改jmxremote.access和jmxremote.password的权限：sudo chmod 600 jmx*

### 5.重新启动Tomcat
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以查看log下的日志看是否有错，并解决。

### 6.打开jvisualvm
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.1 添加远程主机 6.2 添加JMX连接 确定连接即可
![连接](http://img.blog.csdn.net/20170809183806389?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaV9hbV90b21hdG8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 补充：
## 监控JVM中运行的Java程序
前提：远程机器需要有Java环境

1.在%JAVA_HOME%\bin目录下创建一个文件：jstatd.all.policy（名字可以变，扩展名不可以变），内容如下：

	grant codebase "file:${java.home}/../lib/tools.jar" {
       permission java.security.AllPermission;
	};
这个文件的作用是让jstatd服务能够读取机器上的java应用程序的运行数据。

2.在%JAVA_HOME%\bin目录下，执行如下命令：

jstatd -J-Djava.security.policy=jstatd.all.policy

3.展开刚所新建的远程主机，就可看到运行在远程机器上的JAVA应用程序了