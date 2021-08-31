---
title: mongoDB getting started
date: 2021-08-31 17:14:42
tags: ['MongoDB']
---

### 切换DB
进入MongoDB shell, 输入`db`查看当前的数据库：
```
db
```
输入`use <db>`切换数据库：
```
use examples
```

### 往集合里填充数据
MongoDB把数据存在集合（collection）中, 相当于关系型数据库中表（table）的概念，插入数据是，如果集合不存在，MongoDb会自动帮你创建。

```
db.inventory.insertMany([
   { item: "journal", qty: 25, status: "A", size: { h: 14, w: 21, uom: "cm" }, tags: [ "blank", "red" ] },
   { item: "notebook", qty: 50, status: "A", size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank" ] },
   { item: "paper", qty: 10, status: "D", size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank", "plain" ] },
   { item: "planner", qty: 0, status: "D", size: { h: 22.85, w: 30, uom: "cm" }, tags: [ "blank", "red" ] },
   { item: "postcard", qty: 45, status: "A", size: { h: 10, w: 15.25, uom: "cm" }, tags: [ "blue" ] }
]);

// MongoDB adds an _id field with an ObjectId value if the field is not present in the document
```

### 查询所有记录
用`db.collection.find()`可以查询一个集合中的所有记录，`find`方法接受一个过滤条件的对象，可以查出指定条件的数据。
```
db.inventory.find({})
db.inventory.find({}).pretty() //在find后面加pretty可以对结果格式化
```

### 指定等值匹配查询
如果要查找比如`field = value`的记录，可以在`find`传入筛选条件参数。
- 查`status=D`的记录：
```
db.inventory.find( { status: "D" } );
```
- 查`qty=0`：
```
db.inventory.find( { qty: 0 } );
```

### 指定返回字段
`find`还可以接受另一个参数`projection`，表示查询结果返回的字段，格式：
- `<field>: 1` 指定返回该字段；
- `<field>: 0` 指定不返回该字段；
```
db.inventory.find( { }, { item: 1, status: 1 } );
```
`_id`是默认返回的字段，如果不需要返回，需要手动指定该字段不返回：
```
db.inventory.find( {}, { _id: 0, item: 1, status: 1 } );
```
