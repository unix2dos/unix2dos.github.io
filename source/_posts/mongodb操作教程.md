---
title: 'mongodb操作教程'
date: 2017-02-13 17:54:46
tags: sql
---


### 数据库操作

+ 创建数据库

```
use study
```

如果你想查看所有数据库，可以使用 show dbs 命令：
刚创建的数据库并不在数据库的列表中， 要显示它，我们需要向 数据库插入一些数据。

+ 查看当前在哪个数据库

```
> db
study
```
<!-- more -->
+ 删除数据库

```
db.dropDatabase()
```
<!-- more -->
### 文档操作

+ 向文档插入东西

```
db.test.insert({"name":"test"})
```

+ 查询文档内容

```
> db.test.find()
{ "_id" : ObjectId("58da1f0e767a1e8a0cedff28"), "name" : "test" }

db.test.find().pretty() //格式化输出
```

+ 删除文档(注意这个删掉, remove是删除数据)

```
db.test.drop()
```


+ 显示文档

```
> show tables;
system.indexes
test
```

+ 更新文档

```
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)

参数说明：
  ● query : update的查询条件，类似sql update查询内where后面的。
  ● update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
  ● upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
  ● multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
  ● writeConcern :可选，抛出异常的级别
```

例子:

```
db.col.insert({
    title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})

db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}})
db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}},{multi:true})
```

>更多实例:

```
只更新第一条记录：
db.col.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );
全部更新：
db.col.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );
只添加第一条：
db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );
全部添加加进去:
db.col.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );
全部更新：
db.col.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );
只更新第一条记录：
db.col.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );
```

+ save文档

```
db.collection.save(
   <document>,
   {
     writeConcern: <document>
   }
)
  ● document : 文档数据。
  ● writeConcern :可选，抛出异常的级别。

以下实例中我们替换了 _id 为 56064f89ade2f21f36b03136 的文档数据： (通过指定id替换文档)
db.col.save({
    "_id" : ObjectId("56064f89ade2f21f36b03136"),
    "title" : "MongoDB",
    "description" : "MongoDB 是一个 Nosql 数据库",
    "tags" : [
            "mongodb",
            "NoSQL"
    ],
    "likes" : 110
})
```

+ remove 文档

```
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
  ● query :（可选）删除的文档的条件。
  ● justOne : （可选）如果设为 true 或 1，则只删除一个文档。
  ● writeConcern :（可选）抛出异常的级别。
```

如果你想删除所有数据，可以使用以下方式（类似常规 SQL 的 truncate 命令）：

```
>db.col.remove({})
>db.col.find()
>
```



### 文档查询
MongoDB 与 RDBMS Where 语句比较
如果你熟悉常规的 SQL 数据，通过下表可以更好的理解 MongoDB 的条件语句查询：

```
操作 格式 范例 RDBMS中的类似语句
等于 {<key>:<value>} db.col.find({"by":"菜鸟教程"}).pretty() where by = '菜鸟教程'
小于 {<key>:{$lt:<value>}} db.col.find({"likes":{$lt:50}}).pretty() where likes < 50
小于或等于 {<key>:{$lte:<value>}} db.col.find({"likes":{$lte:50}}).pretty() where likes <= 50
大于 {<key>:{$gt:<value>}} db.col.find({"likes":{$gt:50}}).pretty() where likes > 50
大于或等于 {<key>:{$gte:<value>}} db.col.find({"likes":{$gte:50}}).pretty() where likes >= 50
不等于 {<key>:{$ne:<value>}} db.col.find({"likes":{$ne:50}}).pretty() where likes != 50
```

+ MongoDB AND 条件
MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，及常规 SQL 的 AND 条件。
语法格式如下：

```
>db.col.find({key1:value1, key2:value2}).pretty()
```

+ MongoDB OR 条件
MongoDB OR 条件语句使用了关键字 $or,语法格式如下：

```
>db.col.find(
   {
      $or: [
	     {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

+ AND 和 OR 联合使用
以下实例演示了 AND 和 OR 联合使用，类似常规 SQL 语句为： 'where likes>50 AND (by = '菜鸟教程' OR title = 'MongoDB 教程')'

```
>db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
```


### 操作符- $type 实例

```
类型 数字 备注
Double 1 
String 2 
Object 3 
Array 4 
Binary data 5 
Undefined 6 已废弃。
Object id 7 
Boolean 8 
Date 9 
Null 10 
Regular Expression 11 
JavaScript 13 
Symbol 14 
JavaScript (with scope) 15 
32-bit integer 16 
Timestamp 17 
64-bit integer 18 
Min key 255 Query with -1.
Max key 127 
```

如果想获取 "col" 集合中 title 为 String 的数据，你可以使用以下命令：

```
db.col.find({"title" : {$type : 2}})
```

+ Limit()方法  
如果你需要在MongoDB中读取指定数量的数据记录，可以使用MongoDB的Limit方法，limit()方法接受一个数字参数，该参数指定从MongoDB中读取的记录条数。

limit()方法基本语法如下所示：

```
>db.COLLECTION_NAME.find().limit(NUMBER)
```

+ Skip() 方法

我们除了可以使用limit()方法来读取指定数量的数据外，还可以使用skip()方法来跳过指定数量的数据，skip方法同样接受一个数字参数作为跳过的记录条数。

skip() 方法脚本语法格式如下：

```
>db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
```

看下面的这个例子, 说明find没有条件, 结果后面字段显示title, _id不显示, 

```
>db.col.find({},{"title":1,_id:0}).limit(1).skip(1)
{ "title" : "Java 教程" }
>
```


+ Sort()方法
在MongoDB中使用使用sort()方法对数据进行排序，sort()方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而-1是用于降序排列。


sort()方法基本语法如下所示：

```
>db.COLLECTION_NAME.find().sort({KEY:1})
```

```
>db.col.find({},{"title":1,_id:0}).sort({"likes":-1})
{ "title" : "PHP 教程" }
{ "title" : "Java 教程" }
{ "title" : "MongoDB 教程" }
>
```

### 索引:
ensureIndex() 方法
ensureIndex()方法基本语法格式如下所示：

```
>db.COLLECTION_NAME.ensureIndex({KEY:1})
```
语法中 Key 值为你要创建的索引字段，1为指定按升序创建索引，如果你想按降序来创建索引指定为-1即可。

ensureIndex() 方法中你也可以设置使用多个字段创建索引（关系型数据库中称作复合索引）。

```
>db.col.ensureIndex({"title":1,"description":-1})
>
```

ensureIndex() 接收可选参数，可选参数列表如下：

```
Parameter Type Description
background Boolean 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为false。
unique Boolean 建立的索引是否唯一。指定为true创建唯一索引。默认值为false.
name string 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。
dropDups Boolean 在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 false.
sparse Boolean 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 false.
expireAfterSeconds integer 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。
v index version 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。
weights document 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。
default_language string 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语
language_override string 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language.
```


在后台创建索引：

```
db.values.ensureIndex({open: 1, close: 1}, {background: true})
```
通过在创建索引时加background:true 的选项，让创建工作在后台执行



### 聚合:
MongoDB中聚合(aggregate)主要用于处理数据(诸如统计平均值,求和等)，并返回计算后的数据结果。有点类似sql语句中的 count(*)。

aggregate() 方法的基本语法格式如下所示：

```
>db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```

```
> db.col.find().pretty()
{
	"_id" : ObjectId("58dc7efc70d2b5b8821b6184"),
	"title" : "MongoDB Overview",
	"description" : "MongoDB is no sql database",
	"by_user" : "w3cschool.cc",
	"url" : "http://www.w3cschool.cc",
	"tags" : [
		"mongodb",
		"database",
		"NoSQL"
	],
	"likes" : 100
}
{
	"_id" : ObjectId("58dc7fcb70d2b5b8821b6185"),
	"title" : "NoSQL Overview",
	"description" : "No sql database is very fast",
	"by_user" : "w3cschool.cc",
	"url" : "http://www.w3cschool.cc",
	"tags" : [
		"mongodb",
		"database",
		"NoSQL"
	],
	"likes" : 10
}
{
	"_id" : ObjectId("58dc7fe370d2b5b8821b6186"),
	"title" : "Neo4j Overview",
	"description" : "Neo4j is no sql database",
	"by_user" : "Neo4j",
	"url" : "http://www.neo4j.com",
	"tags" : [
		"neo4j",
		"database",
		"NoSQL"
	],
	"likes" : 750
}
```

```
> db.col.aggregate([{$group:{_id:"$by_user", num_tutorial:{$sum:1}}}])
{ "_id" : "Neo4j", "num_tutorial" : 1 }
{ "_id" : "w3cschool.cc", "num_tutorial" : 2 }
>
```
以上实例类似sql语句： select by_user, count(*) from mycol group by by_user

//上面的_id不能变, 后面的字段可以自己指定

下表展示了一些聚合的表达式:

```
表达式 描述 实例
$sum 计算总和。 db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}])
$avg 计算平均值 db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}])
$min 获取集合中所有文档对应值得最小值。 db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}])
$max 获取集合中所有文档对应值得最大值。 db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}])
$push 在结果文档中插入值到一个数组中。 db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}])
$addToSet 在结果文档中插入值到一个数组中，但不创建副本。 db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}])
$first 根据资源文档的排序获取第一个文档数据。 db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}])
$last 根据资源文档的排序获取最后一个文档数据 db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}])
```


### 管道

  ● $project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
  
  ● $match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
  
  ● $limit：用来限制MongoDB聚合管道返回的文档数。
  
  ● $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
  
  ● $unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
  
  ● $group：将集合中的文档分组，可用于统计结果。
  
  ● $sort：将输入文档排序后输出。
  
  ● $geoNear：输出接近某一地理位置的有序文档。


1、$project实例

```
db.article.aggregate(
    { $project : {
        title : 1 ,
        author : 1 ,
    }}
 );
```
这样的话结果中就只还有_id,tilte和author三个字段了，默认情况下_id字段是被包含的，如果要想不包含_id话可以这样:

```
db.article.aggregate(
    { $project : {
        _id : 0 ,
        title : 1 ,
        author : 1
    }});
```
```
> db.article.find()
{ "_id" : ObjectId("58dca39270d2b5b8821b6187"), "name" : "liuwei", "age" : 123 }
> db.article.aggregate(     { $project : {      name:1    }}  );
{ "_id" : ObjectId("58dca39270d2b5b8821b6187"), "name" : "liuwei" }
> db.article.find({}, {name:1})
{ "_id" : ObjectId("58dca39270d2b5b8821b6187"), "name" : "liuwei" }
```
2.$match实例

```
db.articles.aggregate( [
                        { $match : { score : { $gt : 70, $lte : 90 } } },
                        { $group: { _id: null, count: { $sum: 1 } } }
                       ] );
```
$match用于获取分数大于70小于或等于90记录，然后将符合条件的记录送到下一阶段$group管道操作符进行处理。

3.$skip实例

```
db.article.aggregate(
    { $skip : 5 });
```
经过$skip管道操作符处理后，前五个文档被"过滤"掉。


### 数据库引用:

DBRef的形式：

```
{ $ref : , $id : , $db :  }
三个字段表示的意义为：
  ● $ref：集合名称
  ● $id：引用的id
  ● $db:数据库名称，可选参数
```

address DBRef 字段指定了引用的地址文档是在 address_home 集合下的 w3cschoolcc 数据库，id 为 534009e4d852427820000002。
以下代码中，我们通过指定 $ref 参数（address_home 集合）来查找集合中指定id的用户地址信息：

```
>var user = db.users.findOne({"name":"Tom Benzamin"})
>var dbRef = user.address
>db[dbRef.$ref].findOne({"_id":(dbRef.$id)})
```


### 使用 explain()
explain 操作提供了查询信息，使用索引及查询统计等。有利于我们对索引的优化。
接下来我们在 users 集合中创建 gender 和 user_name 的索引：

```
db.users.find({gender:"M"},{user_name:1,_id:0}).explain()
```


### 使用 hint()
虽然MongoDB查询优化器一般工作的很不错，但是也可以使用 hint 来强制 MongoDB 使用一个指定的索引。
这种方法某些情形下会提升性能。 一个有索引的 collection 并且执行一个多字段的查询(一些字段已经索引了)。
如下查询实例指定了使用 gender 和 user_name 索引字段来查询：

```
>db.users.find({gender:"M"},{user_name:1,_id:0}).hint({gender:1,user_name:1})
```

