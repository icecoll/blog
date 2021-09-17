---
title: MongoDB CRUD - 查询二
date: 2021-09-17 23:25:29
tags: ['MongoDB']
---
#### 数组相关的查询
##### 准备数据
```
await db.collection('inventory').insertMany([
  {
    item: 'journal',
    qty: 25,
    tags: ['blank', 'red'],
    dim_cm: [14, 21]
  },
  {
    item: 'notebook',
    qty: 50,
    tags: ['red', 'blank'],
    dim_cm: [14, 21]
  },
  {
    item: 'paper',
    qty: 100,
    tags: ['red', 'blank', 'plain'],
    dim_cm: [14, 21]
  },
  {
    item: 'planner',
    qty: 75,
    tags: ['blank', 'red'],
    dim_cm: [22.85, 30]
  },
  {
    item: 'postcard',
    qty: 45,
    tags: ['blue'],
    dim_cm: [10, 15.25]
  }
]);
```
##### 匹配整个数组
注意使用`{key: value}`这种查询是严格匹配的，包括数组的顺序，比如以下查询`tags`是`['red', 'blank']`的数据，顺序、元素个数不对的话都是不匹配的：
```
const cursor = db.collection('inventory').find({
  tags: ['red', 'blank']
});
```
如果只是想查询`tags`包含`red`和`blank`而不管顺序及元素个数的数据，可以使用`$all`操作符：
```
const cursor = db.collection('inventory').find({
  tags: { $all: ['red', 'blank'] }
});
```
##### 查询单个元素
如果要查询数组包含单个元素，条件的`value`值可以传改元素值而不是数组：
```
const cursor = db.collection('inventory').find({
  tags: 'red'
});
```
也可以使用操作符：
查询`dim_cm`数组中包含值大于25的元素的数据：
```
const cursor = db.collection('inventory').find({
  dim_cm: { $gt: 25 }
});
```
##### 对所有元素指定多个条件
可以对数组中所有的元素指定多个条件，可以返回单个元素满足或多个元素组合起来满足条件的数据，比如：
```
const cursor = db.collection('inventory').find({
  dim_cm: { $gt: 15, $lt: 20 }
});
```
以上查询，可以查出：
```
dim_cm: [14, 21] // 14满足小于20，21满足大于15
dim_cm: [16] // 16同时满足条件
```
如果要排除多个元素组合才满足的情况，可以用`elemMatch`:
```
const cursor = db.collection('inventory').find({
  dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } }
});
// 可以查到 dim_cm: [10, 15.25]，
```
##### 查指定位置
用点号加位置可以查指定位置的元素：
```
 // 查第2个元素大于25的数据，注意数组起始位置是0.
const cursor = db.collection('inventory').find({
  'dim_cm.1': { $gt: 25 }
});

```
##### 查数组长度
用`size`可以查数组长度，如查有3个元素的数据：
```
const cursor = db.collection('inventory').find({
  tags: { $size: 3 }
});
```
#### 包含嵌套数据的数组相关查询
##### 数据准备
```
await db.collection('inventory').insertMany([
  {
    item: 'journal',
    instock: [
      { warehouse: 'A', qty: 5 },
      { warehouse: 'C', qty: 15 }
    ]
  },
  {
    item: 'notebook',
    instock: [{ warehouse: 'C', qty: 5 }]
  },
  {
    item: 'paper',
    instock: [
      { warehouse: 'A', qty: 60 },
      { warehouse: 'B', qty: 15 }
    ]
  },
  {
    item: 'planner',
    instock: [
      { warehouse: 'A', qty: 40 },
      { warehouse: 'B', qty: 5 }
    ]
  },
  {
    item: 'postcard',
    instock: [
      { warehouse: 'B', qty: 15 },
      { warehouse: 'C', qty: 35 }
    ]
  }
]);
```
##### 查询数组中的嵌套数据
查询instock包含元素`{ warehouse: 'A', qty: 5 }`的数据：
```
const cursor = db.collection('inventory').find({
  instock: { warehouse: 'A', qty: 5 }
});
```
这种查询方式必须是严格相同的，属性的个数和顺序等是有影响的，比如以下查询和上面结果是不同的：
```
const cursor = db.collection('inventory').find({
  instock: { qty: 5, warehouse: 'A' }
});
```
##### 对单个字段的查询
使用点号加字段名称查询相应字段：
```
// 查任意instock元素的qty<=20的数据
const cursor = db.collection('inventory').find({
  'instock.qty': { $lte: 20 }
});
```
还可以指定数组下标：
```
// 查第一个元素qty<=20的数据
const cursor = db.collection('inventory').find({
  'instock.0.qty': { $lte: 20 }
});
```
##### 多个条件查询
和普通数组类似，也可以查单个数组元素满足条件，或者多个数组元素组合满足条件：
```
//同一数组元素满足条件
const cursor = db.collection('inventory').find({
  instock: { $elemMatch: { qty: 5, warehouse: 'A' } }
});
const cursor = db.collection('inventory').find({
  instock: { $elemMatch: { qty: { $gt: 10, $lte: 20 } } }
});
// 可以多个数组元素组合满足条件
const cursor = db.collection('inventory').find({
  'instock.qty': { $gt: 10, $lte: 20 }
});
const cursor = db.collection('inventory').find({
  'instock.qty': 5,
  'instock.warehouse': 'A'
});
```
