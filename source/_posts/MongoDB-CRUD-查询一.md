---
title: MongoDB CRUD - 查询一
date: 2021-09-17 17:26:45
tags: ['MongoDB']
---

#### 准备数据
先往`inventory`集合插入数据：
```
await db.collection('inventory').insertMany([
  {
    item: 'journal',
    qty: 25,
    size: { h: 14, w: 21, uom: 'cm' },
    status: 'A'
  },
  {
    item: 'notebook',
    qty: 50,
    size: { h: 8.5, w: 11, uom: 'in' },
    status: 'A'
  },
  {
    item: 'paper',
    qty: 100,
    size: { h: 8.5, w: 11, uom: 'in' },
    status: 'D'
  },
  {
    item: 'planner',
    qty: 75,
    size: { h: 22.85, w: 30, uom: 'cm' },
    status: 'D'
  },
  {
    item: 'postcard',
    qty: 45,
    size: { h: 10, w: 15.25, uom: 'cm' },
    status: 'A'
  }
]);
```

#### 查询全部数据
```
const cursor = db.collection('inventory').find({});
// 相当于以下SQL
SELECT * FROM inventory
```
#### 指定等值查询条件
查`status`为`D`的数据
```
const cursor = db.collection('inventory').find({ status: 'D' });
// SQL
SELECT * FROM inventory WHERE status = "D"
```
#### 使用查询操作符
查询`status`是`D`或`A`的数据：
```
const cursor = db.collection('inventory').find({
  status: { $in: ['A', 'D'] }
});
// SQL
SELECT * FROM inventory WHERE status in ("A", "D")
```
#### 多字段`与`查询
查询`status`为`A` **且** `qty`小于30：
```
const cursor = db.collection('inventory').find({
  status: 'A',
  qty: { $lt: 30 }
});
// SQL
SELECT * FROM inventory WHERE status = "A" AND qty < 30
```
#### 多字段`或`查询
查询`status`为`A` **或** `qty`小于30：
```
const cursor = db.collection('inventory').find({
  $or: [{ status: 'A' }, { qty: { $lt: 30 } }]
});
// SQL
SELECT * FROM inventory WHERE status = "A" OR qty < 30
```
#### 同时指定与和或查询
查询`status`为`A`且（`qty`小于30或item以p开头）：
```
const cursor = db.collection('inventory').find({
  status: 'A',
  $or: [{ qty: { $lt: 30 } }, { item: { $regex: '^p' } }]
});
// SQL
SELECT * FROM inventory WHERE status = "A" AND ( qty < 30 OR item LIKE "p%")
```
#### 查询嵌套数据
##### 匹配嵌入/嵌套数据
查`size`为`{ h: 14, w: 21, uom: "cm" }`的数据：
```
const cursor = db.collection('inventory').find({
  size: { h: 14, w: 21, uom: 'cm' }
});
```
在嵌套数据上使用等值查询的话，传入的值必须严格与要查找的值相等，包括属性的顺序，比如下面的查询是查不到数据的：
```
const cursor = db.collection('inventory').find({
  size: { w: 21, h: 14, uom: 'cm' }
});
```
##### 查询嵌套字段
可以是用点号查询嵌套字段，如查询所有`size`的`uom`属性为in的：
```
const cursor = db.collection('inventory').find({
  'size.uom': 'in'
});
```
同样也可以使用操作符，`AND`查询等：
```
const cursor = db.collection('inventory').find({
  'size.h': { $lt: 15 }
});
const cursor = db.collection('inventory').find({
  'size.h': { $lt: 15 },
  'size.uom': 'in',
  status: 'D'
});
```
