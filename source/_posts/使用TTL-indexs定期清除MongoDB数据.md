---
title: 使用TTL indexs定期清除MongoDB数据
date: 2021-09-10 23:31:18
tags: ['MongoDB']
---

TTL索引是特殊的单字段索引，用户从集合中自动移除数据。应用场景如：日志定期删除，有些保存在数据库的有时效性的session信息数据等。
创建TTL索引可以用`createIndex()`方法，传入`expireAfterSeconds`。
比如在`lastModifiedDate`字段对eventlog集合创建TTL索引：
```
db.eventlog.createIndex( { "lastModifiedDate": 1 }, { expireAfterSeconds: 3600 } )
```
还一种方式是`expireAfterSeconds`设置为0，MongoDB会以字段的时间来过期数据：
```
//创建索引
db.log_events.createIndex( { "expireAt": 1 }, { expireAfterSeconds: 0 } )

//插入数据，数据会在July 22, 2013 14:00:00这个时间点自动被删除
db.log_events.insert( {
   "expireAt": new Date('July 22, 2013 14:00:00'),
   "logEvent": 2,
   "logMessage": "Success!"
} )
```
### 相关行为
#### 数据过期
正常情况下，建立了TTL索引的话，数据的过期时间点就是`被建立索引的字段`expireAfterSeconds`，但是也可能有以下情况：
1. 字段的类型是数组，并且数组中有多个日期值，MongoDB会用最小的值来计算过期时间，即哪个最早用哪个；
2. 如果字段不是日期，或者是数组的时候，没有包含是日期类型的值，那这条记录就不会过期；
3. 如果某条记录不包含创建索引的字段，那这条记录不会过期。比如在`lastModifiedDate`上创建了索引，但是插入数据的时候，没有给`lastModifiedDate`赋值。

#####  删除操作
`mongod`有一个后台线程会读取索引中的值并且清除过期的数据，当TTL线程活动的时候，可以通过`db.currentOp()`看到删除操作，或者通过`Database Profiler`看到。
#### 删除时间
MongoDB从TTL索引在主库上完成建立就开始删除过期数据，关于索引创建过程，请（参考)[https://docs.mongodb.com/v5.0/core/index-creation/#std-label-index-operations]。
TTL索引不保证数据已过期就立即删除，从数据过期到删除这段时间可能会有些许延迟。
删除数据的后台任务每60秒执行一次，所以数据可能会在过期时间到任务执行这段期间仍然存在数据集合中。
因为删除所用的时间取决于`mongod`的工作负荷情况，所以延迟也有可能会超过60秒这个时间段。
##### 主从数据库
TTL后台线程只有在主库状态时才执行删除，在从库状态下回挂起，从库的删除复制主库的删除操作。
#### 对查询的支持
TTL索引对查询的支持和非TTL索引一样
### 局限性
- 只对单字段索引有效，多字段索引会忽略`expireAfterSeconds`.
- `_id`字段不支持。
- `capped collection`不支持，因为不能从`capped collection`删除数据。
- `time series collection`不支持，因为有类似的功能，(参考)[https://docs.mongodb.com/v5.0/core/timeseries/timeseries-automatic-removal/#std-label-manual-timeseries-automatic-removal]
- 不能使用`createIndex()`修改已创建的索引的过期时间，可以使用` collMod`数据库命令结合`index`修改，否则只能删除索引再重新创建。
- 如果一个非TTL索引已经建立了索引，那不能再同时在这个字段上创建TTL索引，只能删除已存在的索引，再重新创建并带上`expireAfterSeconds`。
