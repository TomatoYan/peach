---
title: MongoDB学习(一)
tags:
  - mongoDB
  - 读书笔记
abbrlink: 60989
date: 2017-08-28 13:14:52
---
# 基本概念
## 文档
	文档是MongoDB中数据的基本单元，类似于关系数据库中的行。
	集合可以被看做是没有模式的表。
	MongoDB的单个实例可以容纳多个独立的数据库，每一个都有自己的集合和权限。
	每一个文档都有一个特殊的键 “_id” ，它在文档所处的集合中是唯一的。

## 文档概念
  
- 文档中的键 / 值对是有序的，上面的文档和下面的文档是完全不同的；
- 文档的键是字符串，键不能含有 \0 ，这个字符用来表示键的结尾；
- MongoDB不但区分类型，还区分大小写；
- MongoDB不允许有重复的键；
- 集合就是一组文档；
- 集合是无模式的；

<!-- more -->

## 集合概念
- 集合名不能是空字符串 “”；
- 集合名不能含有 \0 字符串，这个字符表示集合名的结尾；
- 集合名不能以“system”开头，这是为系统保留的前缀；
- 用户创建的集合名字不能含有保留字符 $ ；

## 数据库概念
MongoDB中多个文档组成集合，同样多个集合组成数据库。一个MongoDB实例可以承载多个数据库，每个数据库拥有0个或多个集合，它们之间可以视为完全独立的。每个数据库有独立的权限控制。
数据库名满足的条件：

- 不能是空字符串
- 不能含有空格，. ，$，/，\，\0
- 应全部小写
- 最多64字节

特殊作用的数据库：

- dmin：从权限看，这是“root”数据库，一些特定的服务端命令也只能从这个数据库运行。
- local：这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
- config：当Mongo用于分片设置时，cnfig数据库在内部使用，用于保存分片的相关信息。

启动MongoDB，由于MongoDB一般作为网络服务器来运行，客户端可以连接到该服务器并执行操作，要启动该服务器，需要运行mongod可执行文件。mongod在没有参数的情况下会使用默认数据目录/data/db，并使用27017端口，如果目录不存在或不可写，服务器会启动失败，如果端口被占用也会启动失败。
MongoDB会自带一个JavaScript shell，可以从命令行与MongoDB实例交互。运行mongo启动shell。
## Shell的基本操作
创建、读取、更新、删除(CRUD)。
### 1. 创建
insert函数添加一个文档到集合当中。

	post = {
	"title": "My Blog post",
	"content": "My content",
	"date": "2014 4 4 14:44:44"
	}
post对象是一个有效的MongoDB文档，所以可以采用insert保存到blog集合中：
db.blog.insert(post)
### 2. 读取
find会返回集合里面的所有文档，若只想查看一个文档，可以选择findOne：
db.blog.findOne()
使用find时，shell自动显示最多20个匹配文档。
db.blog.find()
### 3. 更新
update接受(至少)两个参数：第一个是要更新文档的限定条件，第二个是更新的文档。
假设，我们需要在先前的文章增加评论内容，则需要增加一个新的键，
post.comments = []
然后执行update操作，用新版的文档替换标题为“My Blog post”的文章

	db.blog.update({title: "My Blog post"}, post)
	{
        "_id" : ObjectId("59a622650eac87b27720f544"),
        "title" : "My Blog post",
        "content" : "My content",
        "date" : "2014 4 4 14:44:44",
        "comments" : [ ]
	}
### 4. 删除
remove用来从数据库中永久地删除文档，在不使用参数进行调用的情况下，它会删除一个集合内的所有文档。它也可以接受一个文档以指定限定条件。

	db.blog.remove({title: "My Blog post"})
### 注意：
在shell中，只有db中找不到指定的属性时，才会将其作为集合返回。
例如version是数据库函数，

	> db.version
	function(){
	return this.serverBuildInfo.version;
	}
当有属性值与目标集合同名时，可以使用getCollection函数：
	> db.getCollection("version");
	test.version
	db.getCollection("version").find() 便会返回集合version中的数据
## 数据类型：
### 基本数据类型：
MongoDB的文档类似于JSON，在概念上与JavaScript中的对象相似。
JSON仅包含6种数据类型，null、布尔、数字、字符串、数组、对象。
MongoDB在保留JSON基本键值对的基础上，添加了其他的一些数据类型，

- null 用来表示空值或者不存在的字段   {"x": null}
- 布尔 有两个值 'true' 和 'false'   {"x": true}
- 32 位整数 shell中这个类型不可用，因为JavaScript仅支持64位浮点数，所以32位整数会被自动转换
- 64位整数 shell中不支持这个类型，shell会使用一个特殊的内嵌文档来显示64位整数
- 64位浮点数 shell中的数字都是这个类型
- 字符串 UTF-8字符串都可以表示为字符串类型的数据
- 符号 shell不支持这种类型，shell将数据库里的符号类型转换成字符串
- 对象id 对象id是文档的12字节的唯一ID   {"x": ObjectId()}
- 日期 日期类型存数的时从标准纪元开始的毫秒数，不存储时区   {"x": new Date()}
- 正则表达式 采用JavaScript的正则表达式的语法   {"x": /footbar/i}
- 代码 文档中可以有JavaScript代码   {"x": function{/* ... */}}
- 二进制数据 二进制数据可以由任意字节的串组成，shell中没有这个类型
- 最大值 BSON包含一个特殊类型，表示可能的最大值，shell中没有这个类型
- 最小值 BSON包含一个特殊类型，表示可能的最小值，shell中没有这个类型
- 未定义 文档中也可以用未定义类型(JavaScript中null和undefined是不同的类型)   {"x": undefined}
- 数组 值的集合或者列表可以表示成数组   {"x": ["a", "b", "c"]}
- 内嵌文档 文档可以包含别的文档，也可以作为值嵌入到父文档中   {"x": {"hello": "hi"}}

### 注意：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于JavaScript中只有一种数字类型，默认情况下shell中的数字都被MongoDB当作是双精度数。这意味着从数据库中获得的是一个32位整数，修改文档后，当文档返回数据库时，这个整数也被转换成了浮点数。
在JavaScript中，Date对象用作MongoDB的日期类型，创建一个新的Date对象时，通常调用new Date()，而不只是Date()，调用构造函数实际返回的是对日期的字符串表示，而不是真正的Date对象，如果忘记使用Date构造函数，就会造成日期和字符串的混淆。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MongoDB中存储的文档必须有一个"_id"键。这个键的值可以是任何类型的，默认是个ObjectId对象，在一个集合里，每个文档都有唯一的“_id”值。ObjectId使用12字节的存储空间，每个字节两位十六进制数字，是一个24位的字符串。12字节按照如下方式生成：

	0 |1 |2 |3 |4 |5 |6 |7 |8 |9 |10 |11
	时间戳      | 机器   |PID  | 计数器
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;前4个字节是从标准纪元开始的时间戳，单位为秒。由于时间戳在前，这意味着ObjectId会大致按插入顺序排序。
接下来3个字节是所在主机的唯一标识符，通常是机器主机名的散列值，保证了不同主机生产不同的ObjectId
为了保证同一台机器上并发的多个进程产生的ObjectId是唯一的，之后的2字节来自产生ObjectId的进程标识符(PID)
前9字节保证了同一秒钟不同机器不同进程产生的ObjectId是唯一的。后3个字节是一个自动增加的计数器，确保相同进程同一秒产生的ObjectId也是不一样的。
同一秒钟最多允许每个进程拥有 256^3（16777216）个不同的ObjectId。

### 自动生成_id
如果插入文档的时候并没有 _id 键，系统会自动创建一个，这个操作通常会在客户端由驱动程序完成。

- 虽然ObjectId设计为轻量型，但生成毕竟是还是会产生开销。在客户端生成体现了MongoDB的设计理念，能从服务器端转移到驱动程序来做的事，就尽量转移。将事务交由客户端来处理，就减轻了数据库扩展的负担。
- 在客户端生成ObjectId，驱动程序能够提供更加丰富的API。
## 使用MongoDBshell
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;启动mongo shell时不连接到任何mongod，可以通过 --nodb 参数启动。启动之后，在需要时运行new Mongo(hostname) 命令就可以连接到想要的mongod了。

	> conn = new Mongo("localhost:27017")   // 连接本地操作
	connection to localhost:27017
	> db = conn.getDB("test")   // test 为数据库名
	test

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;shell内置了帮助文档，可以使用help命令来查看。可以通过 db.help() 查看数据库级别的帮助。
### 使用shell执行脚本
	$ mongo script1.js script2.js script3.js
	mongo shell会依次执行传入的脚本，然后退出。
	如果希望指定主机/端口上的mongod运行脚本，需要先指定地址，然后再跟上脚本文件的名称：$ mongo --quiet localhost:27017/test script1.js
	这样可以将db指向 localhost:27017上的test数据库，然后运行 script1.js 脚本。
	也可以用load()函数，从交互式shell中运行脚本：
	> load("script1.js")
	shell辅助函数对应的JavaScript函数
	辅助函数等价函数use 数据库名db.getSisterDB("数据库名")show dbsdb.getMongo().getDBs()show collectionsdb.getCollectionNames()

### 创建 .mongorc.js 文件
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果某些脚本会被频繁加载，可以将它们添加到 mongorc.js 文件中。这个文件会在启动shell时自动运行。

	例如：我们希望启动成功时让shell显示一句欢迎语，为此我们在用户主目录下创建一个名为
	.mongorc.js 的文件，向其中添加如下内容：
	// mongorc.js
	var compliment = ["atteactive", "intelligent", "like Batman"];
	var index = Math.floor(Math.random()*3);
	print("Hello, you're looking particularly "+compliment[index]+" today!")
	然后，当启动shell时，就会看到这样一些内容：
	$ mongo
	MongoDB shell version v3.4.4
	Hello, you're looking particularly like Batman today！

### 定制shell提示
	在 .mongorc.js 文件中将prompt变量设为一个字符串或者函数，就可以重写默认的shell提示。
	显示当前时间：
	prompt = function(){
	return (new Date())+">";
	};
	提示当前使用的数据库：
	prompt = function(){
	if(typeof db == 'undefined'){
	return '(nodb)>';
	}
	// 检查最后的数据库操作
	try{
	db.runCommand({getLastError:1});
	}catch(e){
	print(e);
	}
	return db+">";
	};


<!- end ->

<br>
<br><br><br><br><br><br>
<br>
读书笔记 --- MongoDB权威指南 转载表明出处 TomatoYan
<!--戎戎 我喜欢你-->