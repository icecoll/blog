---
title: MongoDB CRUD - 查询三
date: 2021-09-20 22:58:49
tags: ['MongoDB']
---

#### 返回指定字段
MongoDB查询默认返回所有字段，如果要返回指定的字段，可以在查询是加上一个`projection`方法。
##### 准备数据
```
await db.collection('inventory').insertMany([
  {
    item: 'journal',
    status: 'A',
    size: { h: 14, w: 21, uom: 'cm' },
    instock: [{ warehouse: 'A', qty: 5 }]
  },
  {
    item: 'notebook',
    status: 'A',
    size: { h: 8.5, w: 11, uom: 'in' },
    instock: [{ warehouse: 'C', qty: 5 }]
  },
  {
    item: 'paper',
    status: 'D',
    size: { h: 8.5, w: 11, uom: 'in' },
    instock: [{ warehouse: 'A', qty: 60 }]
  },
  {
    item: 'planner',
    status: 'D',
    size: { h: 22.85, w: 30, uom: 'cm' },
    instock: [{ warehouse: 'A', qty: 40 }]
  },
  {
    item: 'postcard',
    status: 'A',
    size: { h: 10, w: 15.25, uom: 'cm' },
    instock: [
      { warehouse: 'B', qty: 15 },
      { warehouse: 'C', qty: 35 }
    ]
  }
]);
```
##### 返回所有字段
```
const cursor = db.collection('inventory').find({
  status: 'A'
});
// SQL
SELECT * from inventory WHERE status = "A"
```
##### 指定返回字段
```
const cursor = db
  .collection('inventory')
  .find({
    status: 'A'
  })
  .project({ item: 1, status: 1 });
// SQL
SELECT _id, item, status from inventory WHERE status = "A"
```
注意，`_id`是会默认返回的，如果不要返回的话，需要在project中指定`_id: 0`。
```
const cursor = db
  .collection('inventory')
  .find({
    status: 'A'
  })
  .project({ item: 1, status: 1, _id: 0 });`
// SQL
SELECT item, status from inventory WHERE status = "A"
```
##### 返回指定字段以外的字段
将需要排除的字段指定为0：
```
const cursor = db
  .collection('inventory')
  .find({
    status: 'A'
  })
  .project({ status: 0, instock: 0 });
```
注意，不能同时有字段为0和1，除了排除_id的情况。
```
project({ status: 0, instock: 1 });  //会报错
project({ status: 1, _id: 0 }); // 不会报错
```
##### 嵌套数据指定返回字段
使用点号即可
```
const cursor = db
  .collection('inventory')
  .find({
    status: 'A'
  })
  .project({ item: 1, status: 1, 'size.uom': 1 });

//数组
const cursor = db
  .collection('inventory')
  .find({
    status: 'A'
  })
  .project({ item: 1, status: 1, 'instock.qty': 1 });
```
MongoDB 4.4开始也可以使用`{ item: 1, status: 1, size: { uom: 1 } }`这种形式。
##### 指定返回数组元素
MongoDB提供`$elemMatch, $slice, $`这三个操作符指定需要返回的数组元素，如返回数组最后一个元素：
```
const cursor = db
  .collection('inventory')
  .find({
    status: 'A'
  })
  .project({ item: 1, status: 1, instock: { $slice: -1 } });
```
注意不能像查询一样指定数组下标: `{ "instock.0": 1 }`
#### 查询null或不存在的字段
##### 准备数据
```
await db.collection('inventory').insertMany([{ _id: 1, item: null }, { _id: 2 }]);
```
##### 查询值为null或者字段不存在
```
// 两条记录都会返回
const cursor = db.collection('inventory').find({
  item: null
});
```
##### 过滤字段不存在的数据
```
// 只返回值为null的数据，字段不存在的不返回
const cursor = db.collection('inventory').find({
  item: { $type: 10 }
});
```
##### 只返回字段不存在的数据
```
const cursor = db.collection('inventory').find({
  item: { $exists: false }
});
```


