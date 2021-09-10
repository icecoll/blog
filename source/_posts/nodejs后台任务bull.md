---
title: nodejs后台任务Bull
date: 2021-09-10 21:17:39
tags: ['nodejs']
---

### Bull是什么
Bull是一个基于redis实现后台队列系统的nodejs包。
### 安装
```
npm install bull --save
// 或
yarn add bull
```
Bull依赖redis，所以你需要有redis支持，可以使用docker快速安装使用，Bull会默认连接到`localhost:6379`
### Queue
通过初始化一个`Bull`对象新建一个队列：
```
const myFirstQueue = new Bull('my-first-queue');
```
一个队列对象通常主要有三种不同角色： `job producer`, `job consumer`, `events listener`，尽管一个给定的实例可以同时用于三种角色，但是通常producer和`consumer`会分开成不同的实例，一个给定的队列可以有多个`producer`，多个`counsumer`，多个`listener`，一个重要的概念是：即使在某时刻没有`consumer`，`producer`也可以添加任务到队列中，因为队列提供异步机制，这是Bull会如此强大的特点之一。

相反地，你可以有一个或多个worker从队列中消费任务，队列消费的顺序默认是FIFO(先进先出)，但也可以是LIFO(后进先出)，或者根据设定的优先级。

关于worker，可以是在同一个也可以是不同进程，可以在同一个机器或者集群，只要保证redis可以同时被`consumer`和`producer`访问到，它们就能协作处理任务。
### Producer
`producer`负责将需要执行的任务塞进队列中，如：
```
const myFirstQueue = new Bull('my-first-queue');
const job = await myFirstQueue.add({
  foo: 'bar'
});
```
可以看到，一个job就是一个node对象，这个对象必须是可序列化的，具体一点就是它必须是它必须可以`JSON.stringify`，因为它就是这样被存在redis中的。同时，还可以在job数据后面传一个options object，后面会讲

### Consumers
`consumer`或者说`worker`是一个需要定义一个process方法的node程序：
```
const myFirstQueue = new Bull('my-first-queue');

myFirstQueue.process(async (job) => {
  return doSomething(job.data);
});
```
`process`方法会在worker闲置并且队列中没有任务要处理时被调用，因为job添加到队列时，`consumer`可能是掉线状态，所以有可能队列中堆积了一堆的job，这时`consumer`进程启动，然后就会一直忙于一个一个去执行队列中的任务，直到所有任务执行完。
上面的例子中，使用了`async`来定义`process`方法，强烈推荐用这种方法定义，如果你的node运行版本不支持`async`，也可以在方法的最后返回一个Promise来达到相同的效果。
process方法的返回值会存在job对象中，以便后续访问，比如在listener的completed事件中。
有时你可能会需要提供任务的处理进度，你可以直接用progress方法：
```
myFirstQueue.process( async (job) => {
  let progress = 0;
  for(i = 0; i < 100; i++){
    await doSomething(job.data);
    progress += 10;
    job.progress(progress);
  }
});

```

### Listener
你可以监听队列中的事件. `listener`可以是局部的，监听指定的队列实例的时间，也可以是全局的，监听所有队列上的事件。所以，你可以在任意实例上添加监听者，不管是`consumer`还是`producer`，但是要注意，如果队列不是`consumer`或者`producer`，那局部事件永远不会触发，这种情况下，你需要是监听全局事件。
```
const myFirstQueue = new Bull('my-first-queue');

// Define a local completed event
myFirstQueue.on('completed', (job, result) => {
  console.log(`Job completed with result ${result}`);
})
```
### job的生命周期
为了充分使用好Bull，理解job的生命周期是很重要的。从`producer`调用add方法添加一个任务到队列中开始，一个job就会开始它的生命周期，期间它会处于不同的状态中，直到完成或者执行失败（执行失败会进入新的生命周期），

当一个job被加入queue之后，它会处于`wait`或者`delayed`状态。job有以下状态：
- wait状态： 所有任务在执行之前都会进入等待列表；
- delayed状态： 表示job在等待达到设定的执行时间，该状态下的job不会直接执行，而是会被丢到等待列表的前面再执行。
- active状态：表示job正在执行，job会一直在active状态，知道完成或者抛出异常。
- completed: 执行完成。
- failed: 执行出错。
### Stalled jobs
Bull定义了一个`Stalled jobs`的概念，`Stalled job`表示这个job被执行过，但是Bull认为它挂了。这种情况发生在这个job在跑`process`方法，并且一直让CPU处于忙碌状态，无法判断是否真的还在执行。
发生这种情况时，Bull会根据这个job的设定，重新让其它的worker执行或者是直接让它进入failed。避免的方法是，不要让node进入太长时间的循环（Bull默认是以几秒钟来判断），或者是用一个独立的` sandboxed processor`。

### 事件
Bull的队列有一堆很实用的事件，事件可以是局部的，也可以是对整个队列实例，例如一个job在某个worker上完成了，针对这个worker的一个局部事件会触发。但是，如果要监听所有的完成事件也是可以的，只要加一个`global`的前缀就行：
局部事件：
```
queue.on('completed', job => {
  console.log(`Job with id ${job.id} has been completed`);
})
```
全局事件：
```
queue.on('global:completed', jobId => {
  console.log(`Job with id ${jobId} has been completed`);
})
```
支持的事件：
```
.on('error', function(error) {
  // An error occured.
})

.on('waiting', function(jobId){
  // A Job is waiting to be processed as soon as a worker is idling.
});

.on('active', function(job, jobPromise){
  // A job has started. You can use `jobPromise.cancel()`` to abort it.
})

.on('stalled', function(job){
  // A job has been marked as stalled. This is useful for debugging job
  // workers that crash or pause the event loop.
})

.on('progress', function(job, progress){
  // A job's progress was updated!
})

.on('completed', function(job, result){
  // A job successfully completed with a `result`.
})

.on('failed', function(job, err){
  // A job failed with reason `err`!
})

.on('paused', function(){
  // The queue has been paused.
})

.on('resumed', function(job){
  // The queue has been resumed.
})

.on('cleaned', function(jobs, type) {
  // Old jobs have been cleaned from the queue. `jobs` is an array of cleaned
  // jobs, and `type` is the type of jobs cleaned.
});

.on('drained', function() {
  // Emitted every time the queue has processed all the waiting jobs (even if there can be some delayed jobs not yet processed)
});

.on('removed', function(job){
  // A job successfully removed.
});
```

### 队列可选配置
#### Rate Limiter
限制执行频率，达到最大限制会进入delayed队列
```
// 限制每5秒最多执行1000个job
const myRateLimitedQueue = new Queue('rateLimited', {
  limiter: {
    max: 1000,
    duration: 5000
  }
});
```
#### Named jobs
给任务命名，可以方便在一些UI工具上查看。
```
// Jobs producer
const myJob = await transcoderQueue.add('image', { input: 'myimagefile' });
const myJob = await transcoderQueue.add('audio', { input: 'myaudiofile' });
const myJob = await transcoderQueue.add('video', { input: 'myvideofile' });
// Worker
transcoderQueue.process('image', processImage);
transcoderQueue.process('audio', processAudio);
transcoderQueue.process('video', processVideo);
```
注意每一个队列实例需要为命名的任务提供一个处理器，否则会报错。
### Sandboxed Processors
如前面介绍，当定义一个`process`方法，可以提供一个并发设置，这个设置允许worker并行处理多个任务，这些任务同时运行在同一个进程上，如果这个任务是IO密集型的任务，那他们可以被处理得很好。
但有时候，有些任务是CPU密集型的，它们会长时间锁住node的事件循环，这会导致Bull将他们置为`stalled`，为了避免这种情况，可以让`process`方法运行在隔离的node进程，用`concurrency`参数控制运行的最大进程数。
我们把这种进程称为`sandboxed process`，它们如果挂了，不会影响其他的进程，并且会自动派生出一个新的进程取代它。
### Job类型
Bull默认的任务类型是FIFO，也就是先来先处理，还有一些其他的类型。
#### LIFO
后来先处理
```
const myJob = await myqueue.add({ foo: 'bar' }, { lifo: true });
```
#### Delayed
可以设置任务等待一段时间再执行，`delay`参数设置等待时间，时间到了会立即被丢到执行队列的前面，只要worker有空闲就会立即执行它。
```
// Delayed 5 seconds
const myJob = await myqueue.add({ foo: 'bar' }, { delay: 5000 });
```
#### Prioritized
可以给任务设置一个优先级参数，优先级高的会先执行，注意最高是1，数值越低，优先级越低。优先队列比普通队列会稍慢，会有一个O(n)的插入时间，n为任务数量，而普通队列是O(1)。
```
const myJob = await myqueue.add({ foo: 'bar' }, { priority: 3 });
```
#### Repeatable
重复队列会重复执行任务，直到达到给定的时间或数值。
```
// Repeat every 10 seconds for 100 times.
const myJob = await myqueue.add(
  { foo: 'bar' },
  {
    repeat: {
      every: 10000,
      limit: 100
    }
  }
);

// Repeat payment job once every day at 3:15 (am)
paymentsQueue.add(paymentsData, { repeat: { cron: '15 3 * * *' } });

```
