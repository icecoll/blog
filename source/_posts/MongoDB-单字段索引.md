---
title: MongoDB 单字段索引
date: 2021-09-11 00:30:09
tags: ['MongoDB']
---
MongoDB支持在集合(collection)的任意字段上建立索引。默认情况下，所以的集合都会有`_id`这个字段的索引，用户可以自己在其他的字段上建立索引，以支持重要的查询与操作。
#### 在单字段上创建升序索引
假如有一个叫`records`的集合有类似这样的数据：
```
{
  "_id": ObjectId("570c04a4ad233577f97dc459"),
  "score": 1034,
  "location": { state: "NY", city: "New York" }
}
```
以下操作可以在score上创建升序索引：
```
db.records.createIndex( { score: 1 } )
```
createIndex接收一个对象，key是字段名称，value表示创建的索引类型，比如`1`表示创建升序索引，`-1`表示创建降序索引。
创建索引后，当执行类似下面的查询，就可以使用索引了：
```
db.records.find( { score: 2 } )
db.records.find( { score: { $gt: 10 } } )
```
#### 在嵌入字段上创建索引
你可以像在第一级字段上创建索引一样在嵌入字段上创建索引。嵌入字段上的索引和嵌入文档的索引有所不同，可以直接用点号。
还是用上面的例子：
```
{
  "_id": ObjectId("570c04a4ad233577f97dc459"),
  "score": 1034,
  "location": { state: "NY", city: "New York" }
}
```
在`location.state`上创建索引：
```
db.records.createIndex( { "location.state": 1 } )
```
然后就可以支持一下查询是使用：
```
db.records.find( { "location.state": "CA" } )
db.records.find( { "location.city": "Albany", "location.state": "NY" } )
```
#### 在嵌入文档(document)上面创建索引
可以直接在整个嵌入文档上创建索引
```
{
  "_id": ObjectId("570c04a4ad233577f97dc459"),
  "score": 1034,
  "location": { state: "NY", city: "New York" }
}

db.records.createIndex( { location: 1 } )
```
查询：
```
db.records.find( { location: { city: "New York", state: "NY" } } )
```
但是要注意，尽管上面的查询可以用到索引，但是返回结果其实并不会包含例子中的数据，因为查嵌入数据时，字段顺序是有影响的。

