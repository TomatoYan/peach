---
title: Nginx负载均衡的搭建
tags: Nginx
abbrlink: 62737
date: 2017-08-02 10:56:18
---

## Nginx的简介
Nginx 是一个很强大的高性能Web和反向代理服务器。

## Nginx优点
1. 作为 Web 服务器：相比 Apache，Nginx 使用更少的资源，支持更多的并发连接，体现更高的效率。在高连接并发的情况下，Nginx是Apache服务器不错的替代品，能够支持高达 50000 个并发连接数的响应。
2. Nginx 配置简洁, Apache 复杂 ，Nginx 启动特别容易, 并且几乎可以做到7*24不间断运行，即使运行数个月也不需要重新启动. 你还能够不间断服务的情况下进行软件版本的升级 . Nginx 静态处理性能比 Apache 高 3倍以上 ，Apache 对 PHP 支持比较简单，Nginx 需要配合其他后端来使用 ,Apache 的组件比 Nginx 多. 
3. 最核心的区别在于apache是同步多进程模型，一个连接对应一个进程；Nginx是异步的，多个连接（万级别）可以对应一个进程 .
4. Nginx的优势是处理静态请求，cpu内存使用率低，apache适合处理动态请求，所以现在一般前端用Nginx作为反向代理抗住压力，apache作为后端处理动态请求。

<!-- more -->

## Nginx 工作原理
Nginx会按需同时运行多个进程：一个主进程(master)和几个工作进程(worker)，配置了缓存时还会有缓存加载器进程(cache loader)和缓存管理器进程(cache manager)等。所有进程均是仅含有一个线程，并主要通过“共享内存”的机制实现进程间通信。主进程以root用户身份运行，而worker、 cache loader和cache manager均应以非特权用户身份运行。

## Nginx的使用
目标：利用Nginx服务器反向代理对Tomcat进行负载均衡
需要的工具：Nginx、Tomcat
负载均衡：

### Nginx的安装
1. 首先下载 Nginx (http://nginx.org/en/download.html)在Linux上解压 tar zxvf nginx-0.x.xx.tar.gz
进入解压目录：cd nginx-0.x.xx
./configure
make
sudo make install
2. Windows只需要解压即可完成安装

### Nginx的启动
以上安装 nginx默认解压在 usr/local/nginx目录下。
所以启动的时候：usr/local/nginx/sbin/nginx -c  /usr/local/nginx/conf/nginx.conf
-c在这里是指定配置文件的路径，如果不指定，那么就是默认的
/usr/local/nginx/conf/nginx.conf
启动后我们可以通过 ps -ef | grep nginx 来查找Nginx的主进程号
master process代表住进程

Windows下直接点击解压包中的nginx.exe 

### Nginx的停止
1) 从容停止Nginx
kill - QUIT <Nginx 主进程号>
或 kill - QUIT '/usr/local/nginx/logs/nginx.pid'
2) 快速停止Nginx
kill - TEAM <Nginx 主进程号>
kill - TEAM Nginx '/usr/local/nginx/logs/nginx.pid'
或
kill - INT <Nginx 主进程号>
kill - INT '/usr/local/nginx/logs/nginx.pid'
3) 强制停止所有Nginx进程
pkill -9 nginx
4) Nginx的平滑重启(因为是从容地重启，因此服务是不中断的)
kill - HUP <Nginx 主进程号>
kill - HUP '/usr/local/nginx/logs/nginx.pid'

Windows下退出直接在任务管理器停止进程即可

## 配置Tomcat
由于Nginx做两个tomcat的反向代理和负载均衡，所以需要修改两个Tomcat的配置，只需要将两个Tomcat的端口号修改为指定端口号。
此处有三处需要修改端口号(修改文件为：apache-tomcat-XXX/conf/server.xml)：

	<?xml version='1.0' encoding='utf-8'?>
	<Server port="18005" shutdown="SHUTDOWN">
	  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
	  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
	  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
	  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
	  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
	  <GlobalNamingResources>
	    <Resource name="UserDatabase" auth="Container"
	              type="org.apache.catalina.UserDatabase"
	              description="User database that can be updated and saved"
	              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
	              pathname="conf/tomcat-users.xml" />
	  </GlobalNamingResources>
	  <Service name="Catalina">
	    <Connector port="18080" protocol="HTTP/1.1"
	               connectionTimeout="20000"
	               redirectPort="8443" />
	    <Connector port="18009" protocol="AJP/1.3" redirectPort="8443" />
	    <Engine name="Catalina" defaultHost="localhost">
	      <Realm className="org.apache.catalina.realm.LockOutRealm">
	       <Realm className="org.apache.catalina.realm.UserDatabaseRealm"resourceName="UserDatabase"/>
	      </Realm>
	      <Host name="localhost"  appBase="webapps"
	            unpackWARs="true" autoDeploy="true">
	        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
	               prefix="localhost_access_log" suffix=".txt"
	               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
	      </Host>
	    </Engine>
	  </Service>
	</Server>

一共三处 另一个tomcat改为 28005、28080、28009即可
配置Nginx
注：配置文件是nginx.conf文件

	worker_processes  2;#工作进程的个数，一般与计算机的cpu核数一致，或者是双倍
	events {
	    worker_connections  1024;#单个进程最大连接数（最大连接数=连接数*进程数）
	}
	http {
	    include       mime.types; #文件扩展名与文件类型映射表
	    default_type  application/octet-stream;#默认文件类型
	    sendfile        on;#开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
		keepalive_timeout  65; #长连接超时时间，单位是秒
	    gzip  on;#启用Gizp压缩
		#服务器的集群
	    upstream  netitcast.com {  #服务器集群名字  此处是两个tomcat
			server    127.0.0.1:18080  weight=1;#服务器配置   weight是权重的意思，权重越大，分配的概率越大。
			server    127.0.0.1:28080  weight=2;
		}	
		#当前的Nginx的配置
	    server {
	        listen  80;#监听80端口，可以改成其他端口
	        server_name  localhost;##############	当前服务的域名
		location / {
	            proxy_pass http://netitcast.com;
	            proxy_redirect default;
	        }
	        error_page   500 502 503 504  /50x.html;
	        location = /50x.html {
	            root   html;
	        }
	    }
	}

## 关闭日志操作(可选)
由于我们的服务需要在服务器上长久运行，日志文件的产生会导致服务器硬盘空间的不足，并且会对系统稳定性造成一定影响，所以我们可以选择关掉日志文件。

### Tomcat日志关闭 ###
将server.xml下的

	<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

注释掉即可。

### Nginx日志关闭 ###
开发环境我默认不写日志，即不配置任何access_log
Nginx的http段中，设置access log：access_log off;
其他日志可以百度查找相关方法