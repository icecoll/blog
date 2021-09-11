---
title: MongoDB 组合索引
date: 2021-09-11 11:09:30
tags: ['MongoDB']
---
MongoDB支持组合索引，也就是可以在同一个集合的多个字段上同时创建索引，字段数上限是32，组合索引在用多字段进行查询时可以发挥作用。
#### 创建组合索引
```
db.collection.createIndex( { <field1>: <type>, <field2>: <type2>, ... } )
```
和单字段上创建索引一样，`type`的值决定索引的类型，比如`1`表示升序索引，`-1`表示降序索引。
注意：
> 从MongoDB 4.4起：
> 组合索引可以且仅可以包含一个hashed index field。
> 创建组合索引时，超过一个hashed index field会报错。
> 在MongoDB 4.4之前：不支持hashed index field。

以下面的`products`集合为例：
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
在`itme`和stock上创建升序组合索引：
```
db.products.createIndex( { "item": 1, "stock": 1 } )
```
组合索引创建时的顺序很重要，上面的例子中，数据的排序会优先使用item排序，再以stock排序。
并且索引会同时对前缀字段生效，比如上面的例子在`item`和`stock`上有索引，同时在`item`上也有索引，以下查询都可以用到索引：
```
db.products.find( { item: "Banana" } )
db.products.find( { item: "Banana", stock: { $gt: 5 } } )
```
#### 排序顺序
索引存储对数据的应用时，要么升序要么降序，对单字段索引来说升序降序读不影响，但是对组合索引，就有影响了，因为它直接决定索引是否支持排序操作。
比如`events`集合包含`username`和`date`两个字段，业务场景上可能会要求以username升序，以date降序排列：
```
db.events.find().sort( { username: 1, date: -1 } )
```
也可能要求以username降序，date升序：
```
db.events.find().sort( { username: -1, date: 1 } )
```
以下索引创建方式可以同时满足上述需求：
```
db.events.createIndex( { "username" : 1, "date" : -1 } )
```
但是，这种方式却不支持同时以username升序，date升序的排序：
```
db.events.find().sort( { username: 1, date: 1 } )
```
要满足的话，可以只用索引交叉的方式，后面会讲到。
#### 前缀
索引前缀是指组合索引字段的前面部分子集，比如：
```
{ "item": 1, "location": 1, "stock": 1 }
```
索引前缀：
- `{ item: 1 }`
- `{ item: 1, location: 1 }`
创建组合索引后，索引会同时在索引前缀生效，也就是查询中包含以下字段的都能用到索引：
- item字段
- item和location字段
- item、location和stock字段
以上例子，MongoDB支持在同时查询item和stock使用索引，但是其实只用到item，相比而言没用直接在item和stock上创建组合索引高效。以下查询都**不支持**使用索引：
- location字段
- field字段
- location和stock字段
如果创建了组合索引({a: 1, b: 1})的同时，又对前缀({a:1})创建了索引，如果不是特殊要求，比如唯一索引约束，那其实可以移除对前缀的索引单独创建，因为MongoDB始终会使用组合索引的前缀索引来查询。
#### 索引交叉
从MongoDB 2.6开始，可以使用交叉索引来满足查询，也就是同时使用两个索引。
举例说明，orders集合有下面两个索引：
```
{ qty: 1 }
{ item: 1 }
```
当执行以下查询是，可以同时使用两个索引：
```
db.orders.find( { item: "abc123", qty: { $gt: 15 } } )
```
可以通过`explain()`来查看是否用了索引交叉，结果中会有`AND_SORTED`或`AND_HASH`。
对索引前缀同样适用：
```
{ qty: 1 }
{ status: 1, ord_date: -1 }

db.orders.find( { qty: { $gt: 10 } , status: "A" } )
```
前面讲过索引前缀有一定的局限性，不支持后面的字段的索引查询和排序等，用交叉索引就可以解决这个问题。
```
//组合索引
{ status: 1, ord_date: -1 }

//支持
db.orders.find( { status: { $in: ["A", "P" ] } } )
db.orders.find(
   {
     ord_date: { $gt: new Date("2014-02-01") },
     status: {$in:[ "P", "A" ] }
   }
)
//不支持
db.orders.find( { ord_date: { $gt: new Date("2014-02-01") } } )
db.orders.find( { } ).sort( { ord_date: 1 } )

//分开建索引的话，所有组合都支持
{ status: 1 }
{ ord_date: -1 }
```
##### 交叉索引与排序
交叉索引的排序，只有在查询条件命中索引规则时，才能使用到索引，比如
```
// orders上的索引
{ qty: 1 }
{ status: 1, ord_date: -1 }
{ status: 1 }
{ ord_date: -1 }

//不支持，尽管status上有索引，但是因为查询条件并没有status
db.orders.find( { qty: { $gt: 10 } } ).sort( { status: 1 } )

//支持，因为查询条件可以命中{ status: 1, ord_date: -1 }这个索引
db.orders.find( { qty: { $gt: 10 } , status: "A" } ).sort( { ord_date: -1 } )
```
