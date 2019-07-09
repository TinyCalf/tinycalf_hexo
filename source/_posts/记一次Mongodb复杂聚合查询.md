---
title: 记一次Mongodb复杂聚合查询
date: 2019-08-01 17:21:17
tags:
- mongodb
categories: mongodb
author: TinyCalf
---
> 今天开发中遇到一个比较复杂的聚合查询，我觉得有必要记录一下；当然不会用项目中的例子，我举个差不多的例子来讲。

学生1,2,3参数可以参加1,2,3三个兴趣小组，每个学生可以参加多个兴趣小组，每个学生参加的兴趣小组有课程时间安排，周一到周五五天的某几天可以参加（1-5表示周一到周五）。于是有了下面这个表，sid表示学生学号，gid表示兴趣小组编号，present表示学生需要出席小组的日期；并且日程安排是新登记到数据库的功能，部分兴趣小组还没有这个字段：
<!-- more -->
```javascript
db.groupstudent.insert([
{_id:"1_2", sid: 1, gid:2, present:[2,3,4]},
{_id:"1_3", sid: 1, gid:3, present:[1,4,5]},
{_id:"2_1", sid: 2, gid:1, present:[2,3]},
{_id:"2_2", sid: 2, gid:2, present:[1,4]},
{_id:"3_1", sid: 3, gid:1, present:[4,5]},
{_id:"3_3", sid: 3, gid:3, present:[1,2,3,4]},
{_id:"3_4", sid: 3, gid:4},
{_id:"2_4", sid: 2, gid:4},
]
)
```
ok，现在需求来了，我们要通过这张表整理出每个兴趣小组每天应该出席的学生列表，还没登记该功能的小组不统计，要求的最终效果如下：
```javascript
{ "gid" : 1, "presentStudentsOfDays" : { "4" : [ 3 ], "3" : [ 2 ], "5" : [ 3 ], "2" : [ 2 ] } }
{ "gid" : 2, "presentStudentsOfDays" : { "2" : [ 1 ], "3" : [ 1 ], "4" : [ 2, 1 ], "1" : [ 2 ] } }
{ "gid" : 3, "presentStudentsOfDays" : { "1" : [ 3, 1 ], "5" : [ 1 ], "4" : [ 3, 1 ], "2" : [ 3 ], "3" : [ 3 ] } }
```
好了，接下来我们一步步做聚合。
### 1.筛选出已经登记出席功能的记录
真实项目中的数据库往往非常庞大，所以我们第一步最好用match做一个筛选，因为只有第一步用match可以用到索引，提高查询速度：
```javascript
db.groupstudent.aggregate(
    {$match: {present:{$exists: true}}}
)
```
查询结果为:
```javascript
{ "_id" : "1_2", "sid" : 1, "gid" : 2, "present" : [ 2, 3, 4 ] }
{ "_id" : "1_3", "sid" : 1, "gid" : 3, "present" : [ 1, 4, 5 ] }
{ "_id" : "2_1", "sid" : 2, "gid" : 1, "present" : [ 2, 3 ] }
{ "_id" : "2_2", "sid" : 2, "gid" : 2, "present" : [ 1, 4 ] }
{ "_id" : "3_1", "sid" : 3, "gid" : 1, "present" : [ 4, 5 ] }
{ "_id" : "3_3", "sid" : 3, "gid" : 3, "present" : [ 1, 2, 3, 4 ] }
```
### 2.映射需要处理的字段
正是情况下每条数据可不止这些字段，为了让后面的处理更快，我们只保留需要使用的字段，下面加一层聚合：
```javascript
db.groupstudent.aggregate(
    {$match: {present:{$exists: true}}},
    {$project: {_id: 0,sid:1, gid:1, presentDay:"$present"}}
)
```
查询结果为:
```javascript
{ "sid" : 1, "gid" : 2, "presentDay" : [ 2, 3, 4 ] }
{ "sid" : 1, "gid" : 3, "presentDay" : [ 1, 4, 5 ] }
{ "sid" : 2, "gid" : 1, "presentDay" : [ 2, 3 ] }
{ "sid" : 2, "gid" : 2, "presentDay" : [ 1, 4 ] }
{ "sid" : 3, "gid" : 1, "presentDay" : [ 4, 5 ] }
{ "sid" : 3, "gid" : 3, "presentDay" : [ 1, 2, 3, 4 ] }
```
### 3.分离数组元素
数组元素在聚合中不太好直接处理，所以我们把它分离出来，每个元素单独分离出一条信息，我们使用unwind关键字：
```javascript
db.groupstudent.aggregate(
    {$match: {present:{$exists: true}}},
    {$project: {_id: 0,sid:1, gid:1, presentDay:"$present"}},
    {$unwind:"$presentDay"}
)
```
查询结果为:
```javascript
{ "sid" : 1, "gid" : 2, "presentDay" : 2 }
{ "sid" : 1, "gid" : 2, "presentDay" : 3 }
{ "sid" : 1, "gid" : 2, "presentDay" : 4 }
{ "sid" : 1, "gid" : 3, "presentDay" : 1 }
{ "sid" : 1, "gid" : 3, "presentDay" : 4 }
{ "sid" : 1, "gid" : 3, "presentDay" : 5 }
{ "sid" : 2, "gid" : 1, "presentDay" : 2 }
{ "sid" : 2, "gid" : 1, "presentDay" : 3 }
{ "sid" : 2, "gid" : 2, "presentDay" : 1 }
{ "sid" : 2, "gid" : 2, "presentDay" : 4 }
{ "sid" : 3, "gid" : 1, "presentDay" : 4 }
{ "sid" : 3, "gid" : 1, "presentDay" : 5 }
{ "sid" : 3, "gid" : 3, "presentDay" : 1 }
{ "sid" : 3, "gid" : 3, "presentDay" : 2 }
{ "sid" : 3, "gid" : 3, "presentDay" : 3 }
{ "sid" : 3, "gid" : 3, "presentDay" : 4 }
```
### 4.按兴趣小组及日期分组
逐渐回到正题了，我们最终的结果需要查询每个兴趣小组每天的出席学生，所以自然要按照presentDay和gid分组：
```javascript
db.groupstudent.aggregate(
    {$match: {present:{$exists: true}}},
    {$project: {_id: 0,sid:1, gid:1, presentDay:"$present"}},
    {$unwind:"$presentDay"},
    {$group:{ _id: {gid:"$gid", presentDay:"$presentDay"}, sids:{$addToSet:"$sid"} }}
)
```
$group需要定义_id和聚合方法，所以第一个参数必须是_id,id中包含gid和presentDay，聚合方法可以有很多包括$sum累加和$count计数等等，我们现在要学生列表，所以我们用$addToSet，把学生组合成一个数组。一下是查询结果：
```javascript
{ "_id" : { "gid" : 3, "presentDay" : 3 }, "sids" : [ 3 ] }
{ "_id" : { "gid" : 1, "presentDay" : 5 }, "sids" : [ 3 ] }
{ "_id" : { "gid" : 3, "presentDay" : 1 }, "sids" : [ 3, 1 ] }
{ "_id" : { "gid" : 3, "presentDay" : 5 }, "sids" : [ 1 ] }
{ "_id" : { "gid" : 1, "presentDay" : 3 }, "sids" : [ 2 ] }
{ "_id" : { "gid" : 2, "presentDay" : 4 }, "sids" : [ 2, 1 ] }
{ "_id" : { "gid" : 3, "presentDay" : 2 }, "sids" : [ 3 ] }
{ "_id" : { "gid" : 2, "presentDay" : 3 }, "sids" : [ 1 ] }
{ "_id" : { "gid" : 2, "presentDay" : 2 }, "sids" : [ 1 ] }
{ "_id" : { "gid" : 1, "presentDay" : 4 }, "sids" : [ 3 ] }
{ "_id" : { "gid" : 3, "presentDay" : 4 }, "sids" : [ 3, 1 ] }
{ "_id" : { "gid" : 1, "presentDay" : 2 }, "sids" : [ 2 ] }
{ "_id" : { "gid" : 2, "presentDay" : 1 }, "sids" : [ 2 ] }
```
### 5.为组合每天的列表做准备
现在我们要把同一个兴趣小组的数据再组合在一起才能达到想要的效果：
```javascript
db.groupstudent.aggregate(
    {$match: {present:{$exists: true}}},
    {$project: {_id: 0,sid:1, gid:1, presentDay:"$present"}},
    {$unwind:"$presentDay"},
    {$group:{ _id: {gid:"$gid", presentDay:"$presentDay"}, sids:{$addToSet:"$sid"} }},
    {$project: {_id: false,gid:"$_id.gid", "presentStudentsOfDays": {k:{$toString:"$_id.presentDay"}, v: "$sids"}}}
)
```
这里又用了$project,为了好看顺便把上一步的数据扁平化;另外把presentStudentsOfDays映射成一个对象，包含上一步的presentDay和sids，分别命名为k和v，并且把`$_id.presentDay`转化成了字符串，这么做是为了下下步使用$arrayToObject转换成对象做准备，关于$arrayToObject可以查看[文档](https://docs.mongodb.com/manual/reference/operator/aggregation/arrayToObject/index.html)。
我们看下效果：
```javascript
{ "gid" : 3, "presentStudentsOfDays" : { "k" : "3", "v" : [ 3 ] } }
{ "gid" : 2, "presentStudentsOfDays" : { "k" : "1", "v" : [ 2 ] } }
{ "gid" : 1, "presentStudentsOfDays" : { "k" : "2", "v" : [ 2 ] } }
{ "gid" : 3, "presentStudentsOfDays" : { "k" : "4", "v" : [ 3, 1 ] } }
{ "gid" : 1, "presentStudentsOfDays" : { "k" : "3", "v" : [ 2 ] } }
{ "gid" : 3, "presentStudentsOfDays" : { "k" : "5", "v" : [ 1 ] } }
{ "gid" : 3, "presentStudentsOfDays" : { "k" : "1", "v" : [ 3, 1 ] } }
{ "gid" : 1, "presentStudentsOfDays" : { "k" : "5", "v" : [ 3 ] } }
{ "gid" : 2, "presentStudentsOfDays" : { "k" : "4", "v" : [ 2, 1 ] } }
{ "gid" : 3, "presentStudentsOfDays" : { "k" : "2", "v" : [ 3 ] } }
{ "gid" : 2, "presentStudentsOfDays" : { "k" : "3", "v" : [ 1 ] } }
{ "gid" : 1, "presentStudentsOfDays" : { "k" : "4", "v" : [ 3 ] } }
{ "gid" : 2, "presentStudentsOfDays" : { "k" : "2", "v" : [ 1 ] } }
```
### 6.按gid分组
这回要把多个gid的条目合并了，很简单$group就行，和之前差不多：
```javascript
db.groupstudent.aggregate(
    {$match: {present:{$exists: true}}},
    {$project: {_id: 0,sid:1, gid:1, presentDay:"$present"}},
    {$unwind:"$presentDay"},
    {$group:{ _id: {gid:"$gid", presentDay:"$presentDay"}, sids:{$addToSet:"$sid"} }},
    {$project: {_id: false,gid:"$_id.gid", "presentStudentsOfDays": {k:{$toString:"$_id.presentDay"}, v: "$sids"}}},
    {$group:{ _id: "$gid", presentStudentsOfDays:{$addToSet:"$presentStudentsOfDays"} }}
)
```
查询结果：
```javascript
{ "_id" : 1, "presentStudentsOfDays" : [ { "k" : "4", "v" : [ 3 ] }, { "k" : "3", "v" : [ 2 ] }, { "k" : "5", "v" : [ 3 ] }, { "k" : "2", "v" : [ 2 ] } ] }
{ "_id" : 2, "presentStudentsOfDays" : [ { "k" : "2", "v" : [ 1 ] }, { "k" : "3", "v" : [ 1 ] }, { "k" : "4", "v" : [ 2, 1 ] }, { "k" : "1", "v" : [ 2 ] } ] }
{ "_id" : 3, "presentStudentsOfDays" : [ { "k" : "1", "v" : [ 3, 1 ] }, { "k" : "5", "v" : [ 1 ] }, { "k" : "4", "v" : [ 3, 1 ] }, { "k" : "2", "v" : [ 3 ] }, { "k" : "3", "v" : [ 3 ] } ] }
```
### 7.数组转对象
现在我们已经很接近我们的结果了，只需要把presentStudentsOfDays转换成需要的对象，要用到刚才说的$arrayToObject了：
```javascript
db.groupstudent.aggregate(
    {$match: {present:{$exists: true}}},
    {$project: {_id: 0,sid:1, gid:1, presentDay:"$present"}},
    {$unwind:"$presentDay"},
    {$group:{ _id: {gid:"$gid", presentDay:"$presentDay"}, sids:{$addToSet:"$sid"} }},
    {$project: {_id: false,gid:"$_id.gid", "presentStudentsOfDays": {k:{$toString:"$_id.presentDay"}, v: "$sids"}}},
    {$group:{ _id: "$gid", presentStudentsOfDays:{$addToSet:"$presentStudentsOfDays"} }},
    {$project: {_id:0, gid: "$_id", presentStudentsOfDays:{$arrayToObject:"$presentStudentsOfDays"}}}
)
```
查询结果：
```javascript
{ "gid" : 1, "presentStudentsOfDays" : { "4" : [ 3 ], "3" : [ 2 ], "5" : [ 3 ], "2" : [ 2 ] } }
{ "gid" : 2, "presentStudentsOfDays" : { "2" : [ 1 ], "3" : [ 1 ], "4" : [ 2, 1 ], "1" : [ 2 ] } }
{ "gid" : 3, "presentStudentsOfDays" : { "1" : [ 3, 1 ], "5" : [ 1 ], "4" : [ 3, 1 ], "2" : [ 3 ], "3" : [ 3 ] } }
```
完成！这回查询常用的有$match/$group/$project，不太常用的有$unwind/$addToSet/$arrayToObject。这个结果在代码里基本上可以直接拿来用了，并不需要后端再做什么处理，毕竟代码写起来要比聚合复杂的多，效率也低，堆栈甚至不一定放得下。最后再提一句，真实数据库里数据庞大，在聚合时mongodb内存文档上限时128M，超过这个数值就需要手动调成硬盘去处理啦，代码如下：
```javascript
db.groupstudent.aggregate(
    [各种聚合],
    {
        allowDiskUse: true
    }
)
```
再见！👋
