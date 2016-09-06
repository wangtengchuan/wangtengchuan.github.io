---
layout: post
title: "Introduction to MongoDB"
description: ""
category: database
tags: [database, nosql]
---
{% include JB/setup %}

# Introduction to MongoDB

![mongoDB logo](https://webassets.mongodb.com/_com_assets/global/mongodb-logo-white.png)


## 什么是MongoBD
**MongoDB**是一个基于分布式文件存储的数据库。由C++编写。一般来讲，我们可以选择的数据库有Mysql, MongoDB, Redis.他们的区别是什么？适用场景是什么？本文主要通过讲解MongoDB的基本用法、特性对MongoDB特性、适用场景有基本理解。

## 为什么选择MongoDB
**MongoDB**是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。他支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。MongoDB最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。
适用场景：

* 网站数据：MongoDB非常适合实时的插入，更新与查询，并具备网站实时数据存储所需的复制及高度伸缩性。
* 缓存：由于性能很高，Mongo 也适合作为信息基础设施的缓存层。在系统重启之后，由Mongo 搭建的持久化缓存层可以避免下层的数据源过载。
*  大尺寸、低价值的数据：使用传统的关系型数据库存储一些数据时可能会比较昂贵，在此之前，很多时候程序员往往会选择传统的文件进行存储。
* 高伸缩性的场景：MongoDB 非常适合由数十或数百台服务器组成的数据库，Mongo 的路线图中已经包含对MapReduce引擎的内置支持。
* 用于对象及JSON数据的存储：MongoDB的BSON数据格式非常适合文档化格式的存储及查询。

个人使用下来的感受：

* 当系统内存可用空间充足的时候，mongoDB会将文档加载到内存中(可以配置)。所以检索速度相当快，比单纯的Mysql快一个数量级；
* 强大的查询功能，支持Aggregation以及Map-reduce;
* 灵活的schema(对数据没有非常强的规整性约束)

## 基本概念

### BSON
通俗一点理解，BSON是一种二进制化的json数据。不过不同点在于BSON是有强类型约束的，而json是弱类型。强类型的可序列化数据可以用作RPC, 这点上一般把BSON跟protobuf比较。

####BSON数据类型

Type	|Number
------|-------
Double	|“double”	 
String	|“string”	 
Object	|“object”	 
**Array**|“array”	 
Binary data| “binData”	
ObjectId| “objectId”	 
Boolean|	“bool”	 
Date| “date”	 
Null|	“null”	 
**Regular Expression**|	“regex”	 
DBPointer	|	“dbPointer”	 
JavaScript|“javascript”	 
Symbol|	“symbol”	 
JavaScript (with scope)	|	“javascriptWithScope”
32-bit integer|	“int”	 
Timestamp|“timestamp”	 
64-bit integer|	“long”	 
Min key|	“minKey”	 
Max key|“maxKey”

### 数据模型
databases(可以理解成Mysql的一个数据库)
	|
collection(一个database可以有多个colletion,类比table)
	|
document(一行数据对应一个document，只不过相对于Mysql的一行，document结构比较松散)
{“hello,word”:“Mike”}


### 基本操作

```
db.colletion.insert(
	{
		name: "sue",
		age: 26,
		status: "A"
	}
)

db.user.find(
	{age: {$gt: 18}},
	{name: 1}
).limit(5)

db.user.update(
	{age:{$gt:18}},
	{$set: {status: "A"}}
)

db.user.remove(
	{status: "D"}
)
```

结果

```
{
  _id: ObjectId("509a8fb2f3f4948bd2f983a0"),
  user_id: "abc123",
  age: 55,
  status: 'A'
}
```
那么与Mysql的区别在哪里呢？MongoDB在插入前不需要定义一个表的schema，也就是说文档类型是灵活的。

#### Bulk Write
批量插入，可以减少io次数。同时MongoDB可以指定Bulk Write是有序还是无序的。
不过不同于Mysql的Transaction, MongoDB不能保证Bulk Write的原子性，也就是不能如果多条指令的一条出错，那么这条指令之前的都会被执行，这条之后的不会被执行，不支持回滚。如果你熟悉Redis的话，你也会知道Redis的Transaction也是不能回滚的。

MongoDB Bulk Write

```
db.characters.bulkWrite(
      [
         { insertOne :
            {
               "document" :
               {
                  "_id" : 4, "char" : "Dithras", "class" : "barbarian", "lvl" : 4
               }
            }
         },
         { insertOne :
            {
               "document" :
               {
                  "_id" : 5, "char" : "Taeln", "class" : "fighter", "lvl" : 3
               }
            }
         },
         { updateOne :
            {
               "filter" : { "char" : "Eldon" },
               "update" : { $set : { "status" : "Critical Injury" } }
            }
         },
         { deleteOne :
            { "filter" : { "char" : "Brisbane"} }
         },
         { replaceOne :
            {
               "filter" : { "char" : "Meldane" },
               "replacement" : { "char" : "Tanys", "class" : "oracle", "lvl" : 4 }
            }
         }
      ]
   );

```
Redis Transaction

```
> MULTI
OK
> INCR foo
QUEUED
> INCR bar
QUEUED
> EXEC
1) (integer) 1
2) (integer) 1

```

那么他们与什么区别，我个人理解，Redis的Transaction更像是Mysql的stored procedures.它能保证一系列指令按照顺序执行，但是不能保证原子性。MongoDB同样不能保证原子性，它的执行顺序是可以手动指定的。

#### pipeline
MongoDB的pipeline跟Linux Shell的pipeline相似，上一个的输出作为下一个的输入。

```
db.collection.aggregate([{$match:{status:"A"}},
							{$group:{_id:"$cust_id", 
							total: {$sum: "$amount"}}}
]
)

db.orders.mapReduce(
			function() {emit(this.cust_id, this.amount);},
			function() {key, values} (return Array.sum(values)},
			{
				query:{$or: [{status: "A"}, {age:{$lt: 30}}},
				out: "order_totals"
			}
)
```
query->map->reduce

#### Reference
类似于Mysql的外键，可以用来减少表中的冗余信息。
![reference](https://docs.mongodb.com/manual/_images/data-model-normalized.png)

## Advanced

### Index
MongoDB提供了非常完整而强大的索引功能。默认情况下，对_id字段进行索引(所以_id不能重复)。我们自己也可以为某个字段增加index:

```
{
  "_id": ObjectId("570c04a4ad233577f97dc459"),
  "score": 1034,
  "location": { state: "NY", city: "New York" }
}
```
下面以升序的方式建立索引(-1是降序)

```
db.records.createIndex( { score: 1 } )
```
那么我们在find时候就会用到这个index

```
db.records.find( { score: 2 } )
```
####嵌套的Index
对于嵌套的field,我们既可以对整个嵌套域建立索引，也可以对嵌套域内的某个field建立索引。

```
{
  "_id": ObjectId("570c04a4ad233577f97dc459"),
  "score": 1034,
  "location": { state: "NY", city: "New York" }
}
```
那么可以对location本身建立索引:

```
db.records.createIndex( { location: 1 } )
```
也可以对location中的state字段建立索引。

```
db.records.createIndex( { "location.state": 1 } )
```

####联合索引

MongoDB可以建立不超过31个索引项的联合索引。

```
{
 "_id": ObjectId(...),
 "item": "Banana",
 "category": ["food", "produce", "grocery"],
 "location": "4th Street Store",
 "stock": 4,
 "type": "cases"
}
```

```
db.products.createIndex( { "item": 1, "stock": 1 } )
```
这个联合索引对于下面两种查询都支持：

```
db.products.find( { item: "Banana" } )
db.products.find( { item: "Banana", stock: { gt: 5 } } )
```

### Storage Engine
strorage engine决定了数据在（内存和磁盘中的）存储和管理方式。MongoDB主要有三种strorage engine(具体用哪种可以在启动mongo-server的时候配置，如果不配置的话就用默认的):

#### WiredTiger
MongoDB 3.2开始支持的默认的存储引擎，也是官方推荐的存储引擎。主要功能:

* document-level concurrency
多个客户度可以同时写一个collection的不同的行，与mysql InnoDB行级锁是一样的。
* checkpoingting & journal
checkpoint主要是在插入数据的时候做快照，journal主要是记录所有的数据修改。
* compressing
MongoDB可以利用空闲的CPU对数据和索引进行压缩。默认使用zlib压缩算法
* encryption

#### MMAPv1
MongoDB3.2之前的默认，这里略过。

#### In-Memory Storage Engine

### Replication(复制)
与Mysql, Redis一样，MongoDB支持复制。**复制集**提供了冗余、读写分离、容错、failover以及其他一些高性能。

#### 基本原则
一个**复制集**只能有一个主数据库，只有主数据库节点可以接受写的请求。与Mysql一样，主节点纪录所有的修改到一个log文件中（在Mysql中是binLog）。

#### hearbeat(心跳检测)
如果primary在10s之内没有与其他数据库通讯，就认为primary挂掉了。那么剩下的数据库就会通过一个投票算法选出新的primary，整个过程是自动进行的，无需人工干预。

##一点结论
MongoDB是一个基于文档的数据库，MongoDB善长的是对无模式JSON数据的查询。由于可以放在内存中，查询效率比Mysql高。
而Redis是一个基于内存的Key-Value数据库，先读写内存再异步同步到磁盘，读写速度上比MongoDB有巨大的提升。但是不支持复杂查询。

测试：500W条嵌套式(embedded document)数据，根据子内容条件查询获取父内容的操作只需要一次I/O，平均耗时1ms（未进行cluster，条件字段有索引，整个Collection体积将近10G）。

上述操作对于Memcached＋Mysql组合通常需要2-3次I/O（区别于具体设计），即：通过子内容条件查询到子内容主键，然后从Memcached中缓存的entity获取关联的父内容主键，然后从Memcached中获取父内容。



