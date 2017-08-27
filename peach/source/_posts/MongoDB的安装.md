---
title: MongoDB的安装
abbrlink: 16902
date: 2017-08-23 19:40:57
tags: MongoDB
---

# mongodb Linux下的安装
## 下载mongodb
    wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-amazon-3.4.7.tgz
## 解压mongodb安装包
    tar -zxvf mongodb-linux-x86_64-amazon-3.4.7.tgz
## 将解压后的文件移动到指定目录(可选)
    mv mongodb-linux-x86_64-amazon-3.4.7/  /usr/local/mongodb

<!-- more -->

## 配置环境变量：
    vim ~/.bashrc  添加如下内容：
    export MONGODB_HOME=/usr/local/mongodb
    export PATH=$MONGODB_HOME/bin:$PATH

    配置生效： source ~/.bashrc
    mongod -v 查看mongodb版本信息查看环境变量是否配置正确
## 创建数据库目录：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MongoDB的数据存储在data目录的db目录下，但是这个目录在安装过程不会自动创建，所以你需要手动创建data目录，并在data目录中创建db目录,注意：如果你的数据库目录不是/data/db，可以通过 --dbpath 来指定
mkdir -p /data/db
mkdir -p /data/log
touch /data/log/mongodb.log
### 添加配置文件：
    touch /etc/conf/mongodb.conf
    新建mongodb.conf配置文件
    vim /etc/conf/mongodb.conf
### 配置文件内容
    dbpath=/data/db
    logpath=/data/log/mongodb.log
    logappend=true
    port=27017
    <!-- 配置放在其他位置没有试过 -->
### 配置文件参数说明:
#### mongodb的参数说明：
	--dbpath 数据库路径(数据文件)
	--logpath 日志文件路径
	--master 指定为主机器
	--slave 指定为从机器
	--source 指定主机器的IP地址
	--pologSize 指定日志文件大小不超过64M.因为resync是非常操作量大且耗时，最好通过设置一个足够大的oplogSize来避免resync(默认的 oplog大小是空闲磁盘大小的5%)。
	--logappend 日志文件末尾添加
	--port 启用端口号
	--fork 在后台运行
	--only 指定只复制哪一个数据库
	--slavedelay 指从复制检测的时间间隔
	--auth 是否需要验证权限登录(用户名和密码)
## 通过配置文件启动服务：
    mongod --journal -f /etc/mongodb.conf  --fork (32位系统需要加参数 --journal  fork后台运行服务) 
## 通过配置文件关闭服务：
    mongod --journal --shutdown -f /etc/mongodb.conf
    可以省略为 mongod --shutdown
## 通过命令行启动服务：
    mongod --dbpath=/data/db/ --logpath=/data/log/mongodb.log --fork
## 进入mongodb后台管理shell
    进入bin目录 cd /usr/local/mongodb/bin
    执行   ./mongo
### 创建数据
    命令： use test
    switched to db test
### 创建用户，设置权限

``` db.createUser({ 
	user:"test",<br>
	pwd:"test", <br>
	roles:[{role:"readWrite", db:"test"}]
	}) ```

![这里写图片描述](http://img.blog.csdn.net/20170827185436097?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaV9hbV90b21hdG8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
## mongodb web界面
    mongod -f /etc/mongodb.conf --rest
    在地址栏输入 127.0.0.1:28017     注意，端口号会比默认设定增加100

# Window下的安装
## 首先下载安装包
![这里写图片描述](http://img.blog.csdn.net/20170827185532993?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaV9hbV90b21hdG8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170827185602246?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaV9hbV90b21hdG8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170827185626674?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaV9hbV90b21hdG8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

	此时已经有了一个db的文件夹

## 在Windows下运行MongoDB服务器
    1. 进入MongoDB下的bin目录
    2. 执行mongod.exe 文件
    .\mongod.exe --dbpath "C:\Programs\mongodb_sample\data\db"
    "C:\Programs\mongodb_sample\data\db" 是数据库的文件目录
    3. 执行成功会有如下的输出：
![这里写图片描述](http://img.blog.csdn.net/20170827185657764?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaV9hbV90b21hdG8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

    然后打开新的窗口输入  .\mongo.exe即可进入mongodb服务器
![这里写图片描述](http://img.blog.csdn.net/20170827185825745?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaV9hbV90b21hdG8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
