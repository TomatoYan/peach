---
title: 配置最小oracle实例单元
tags: 数据库
abbrlink: 46172
date: 2017-08-07 15:12:56
---

## 前言

Oracle开发的关系数据库产品因性能卓越而闻名，许多大型网站也选用了Oracle系统，是世界最好的数据库产品。但Oracle数据库管理软件却十分庞大，在本地安装总会开启一堆服务，同时占用大量的磁盘空间和内存。Oracle提供了Oracle SQL Developer这样的管理工具(下载地址: http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html)，我们可以使用这个工具对本地和远程的Oracle数据库进行管理，SQL Developer是Oracle公司出品的一个免费非开源的用以开发数据库应用程序的图形化工具，使用 SQL Developer 可以浏览数据库对象、运行 SQL 语句和脚本、编辑和调试 PL/SQL 语句，但SQL Developer 需要的开发环境要去会比较高(JDK >1.8)，同时，只可以连接到任何Oracle 10g及其后续版本的数据库，此外SQL Developer市场占有率并不高，市场份额基本都被PL/SQL Developer所占有。而PL/SQL Developer 的使用需要有Oracle的一些支持(SQL Developer不需要)，所以我们需要配置最小Oracle实例单元。

<!-- more -->

## 准备工作
### 下载：

	1.需要在Oracle官网下载简易的oracle数据库客户端(instantclient-basiclite-nt-11.20.0.4.0)，大小仅仅10多M。下载instantclient-basic-win32-11.2.0.1.0
	（oracle官网下载地址http://www.oracle.com/technetwork/topics/winsoft-085727.html )
	2.下载PL/SQL客户端(官网(https://www.allroundautomations.com/bodyplsqldevreg.html)试用，也可以下载破解版)

## 配置Oracle监听

解压instantclient-basiclite-nt-11.20.0.4.0 在其文件夹中新建 network 文件夹，新建 admin 文件夹，这个类似oracle数据库的network -> admin，在文件夹中新建 tnsnames.ora 或者从有数据库客户端的地方拷贝。

	TEST =
	  (DESCRIPTION =
	    (ADDRESS_LIST =
	      (ADDRESS = (PROTOCOL = TCP)(HOST = xxx.xxx.xxx.xxx)(PORT = 1521))
	    )
	    (CONNECT_DATA =
	      (SID = test)
	    )
	  )
也可以直接从oracle客户端里面找到tnsnames.ora文件，直接copy到ADMIN文件下。

## 设置PL/SQL Developer

打开PLSQL Developer，选择Tools -> perference -> Connection，配置其中的Oracle Home和OCI Library项，如下图所示：
    

	其中, Oracle Home：E:\app\Administrator\product\instantclient_11_2
    	 OCI Library：E:\app\Administrator\product\instantclient_11_2\oci.dll

## 配置系统环境变量

右击"我的电脑" - "属性" - "高级" - "环境变量" - "系统环境变量":

	1>.选择"Path" - 点击"编辑", 把 "E:\app\Administrator\product\instantclient_11_2;" 加入;
	2>.点击"新建", 变量名设置为"TNS_ADMIN", 变量值设置为"E:\app\Administrator\product\instantclient_11_2;", 点击"确定";
	3>.点击"新建", 变量名设置为"NLS_LANG", 变量值设置为"SIMPLIFIED CHINESE_CHINA.ZHS16GBK", 点击"确定";字符编码变量配置，防止有乱码