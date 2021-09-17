---
title: MongoDB CRUD - 插入
date: 2021-09-17 11:42:34
tags: ['MongoDB']
---

#### 插入单条数据
`Collection.insertOne()`插入单条数据。下面例子往`inventory`插入单条数据，如果没有传`_id`，nodejs会自动生成加上。
```
await db.collection('inventory').insertOne({
  item: 'canvas',
  qty: 100,
  tags: ['cotton'],
  size: { h: 28, w: 35.5, uom: 'cm' }
});
```
`insertOne()`返回的是一个带`result`的Promise。
#### 插入多条数据
MongoDB 3.2以上支持用`Collection.insertMany()`插入多条数据，参数传入一个数组，同样，如果不带_id，nodejs会自动生成。
```
await db.collection('inventory').insertMany([
  {
    item: 'journal',
    qty: 25,
    tags: ['blank', 'red'],
    size: { h: 14, w: 21, uom: 'cm' }
  },
  {
    item: 'mat',
    qty: 85,
    tags: ['gray'],
    size: { h: 27.9, w: 35.5, uom: 'cm' }
  },
  {
    item: 'mousepad',
    qty: 25,
    tags: ['gel', 'blue'],
    size: { h: 19, w: 22.85, uom: 'cm' }
  }
]);
```

#### 插入行为
##### 创建集合(collection)
如果集合不存在，插入操作会导致创建集合。
##### `_id`字段
MongoDB每一条数据都需要一个唯一的`_id`字段，这个`_id`字段会被当成主键。如果插入时没有带_id字段，MongoDB会自动生成一个`ObjectId`给`_id`字段。`创建更新`操作(带`upsert: true`)同样适用。
##### 原子性
MongoDB所有对单条记录的写操作都是原子的。
##### 写应答
详见`write concern`
