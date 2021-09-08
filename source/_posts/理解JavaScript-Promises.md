---
title: 理解JavaScript Promises
date: 2021-09-08 18:20:37
tags: ['nodejs', 'promise', 'javascript']
---

### Promises简介
`promise`通常被定义为：对一个`最终变为可用状态的值`的代理。
Promises是处理异步代码的一种方式，可以让你避免掉入回调的深渊，就像这样：
```
doSomething(function(result) {
  doSomethingElse(result, function(newResult) {
    doThirdThing(newResult, function(finalResult) {
      ...
    }, failureCallback);
  }, failureCallback);
}, failureCallback);

```
Promises早在ES2015就已经是被nodejs所支持，在ES2017支持`async/await`之后更为完善，`async`方法背后其实就是用的Promises，所以理解Promises是理解`async/await`的基础。
### Promises运作简介
一旦一个`promise`被调用，它会立即开始处于一个`pending`状态，这意味调用函数会一直执行着，直到这个Promise从pending到resolved或rejected，给调用函数返回任意数据。
被创建的Promise最终会变成`resolved`或`rejected`状态，并在结束前调用相应的回调函数(`then/catch`里面的callback)。
### 创建一个Promise
Promise api提供一个构造器函数，所以可以使用`new Promise()`来初始化一个Promise：
```
let done = true
const isItDoneYet = new Promise((resolve, reject) => {
  if (done) {
    const workDone = 'Here is the thing I built'
    resolve(workDone)
  } else {
    const why = 'Still working on something else'
    reject(why)
  }
})

```
上面的代码中，Promise判断`done`是否为true，如果是，会调用`resolve`进入`resolved`状态，否者调用`reject`进入`rejected`状态。需要注意的是，如果执行过程中，`resolve`和`reject`都没有调用的话，那这个Promise就会一直处于`pending`状态。

`resolve`和`rejected`可以通知知调用者Promise当前的状态以及接下来要执行的动作，上述代码中，只是返回了一个字符串，也可以返回对象、`null`。需要注意，创建Promise之后，其实它就已经开始执行了。

更常见的一个例子是一直叫做Promise化的技术，这种技术是让接收一个callback的函数，返回一个Promise：
```
const fs = require('fs')

const getFile = (fileName) => {
  return new Promise((resolve, reject) => {
    fs.readFile(fileName, (err, data) => {
      if (err) {
        reject(err)  // calling `reject` will cause the promise to fail with or without the error passed as an argument
        return        // and we don't want to go any further
      }
      resolve(data)
    })
  })
}

getFile('/etc/passwd')
.then(data => console.log(data))
.catch(err => console.error(err))

```

### 使用Promise
上面创建了一个Promise，看看如何使用：
```
const isItDoneYet = new Promise(/* ... 如上 ... */)
//...

const checkIfItsDone = () => {
  isItDoneYet
    .then(ok => {
      console.log(ok)
    })
    .catch(err => {
      console.error(err)
    })
}

```
调`checkIfItsDone`方法，当`isItDoneYet`最终是resolved状态是，那then里面的回调方法就会执行，如果是rejected，那catch里面的回调就会被执行。
### Promises链
一个Promise可以返回给另一个Promise，从而创建Promises链。
Fetch API就是一个Promises链的例子，我们可以用来获取资源并且通过Promises链依次列出资源被获取到之后需要执行的一系列操作。
Fetch API是基于Promise的，调用`fetch()`就相当于用`new Promise()`定义一个Promise。
例子：
```
const status = response => {
  if (response.status >= 200 && response.status < 300) {
    return Promise.resolve(response)
  }
  return Promise.reject(new Error(response.statusText))
}
const json = response => response.json()
fetch('/todos.json')
  .then(status)    // note that the `status` function is actually **called** here, and that it **returns a promise***
  .then(json)      // likewise, the only difference here is that the `json` function here returns a promise that resolves with `data`
  .then(data => {  // ... which is why `data` shows up here as the first parameter to the anonymous function
    console.log('Request succeeded with JSON response', data)
  })
  .catch(error => {
    console.log('Request failed', error)
  })

```
上述例子中，调用`fetch()`从服务器的根路径下获取todos.json，并且创建了一个Promises链，`fetch()`会返回一个响应(response)，这个响应包含很多属性，我们比较关心的有:
- `status`, 服务器返回的响应码，如200，400，500等，
- `statusText`, 状态消息，比如如果是200，就是`OK`。
`response`也有一个`json()`方法，它会返回一个Promise，将response body解析成JSON格式。

上述代码做了什么：首先定义了一个叫`status()`的方法, 检查响应状态，如果不是成功的响应，就会调`Promise.reject()`，这会导致直接跳过所有的链上的其他Promises，而进到`catch()`的回调中，打印出`Request failed`的error；如果是成功响应的话，会调`Promise.resove()`，接着会调链上的下一个Promise，也就是`json()`方法，因为前面`status()`调resolve传入了`response`，所以`json()`方法可以接收它作为入参，再直接返回`response.json()`，最后一个Promise收到这个JSON数据，直接打印出来：
```
.then((data) => {
  console.log('Request succeeded with JSON response', data)
})
```
### 处理error
上面的例子中，在Promise链的最后有一个`catch`，当前面的Promises有任何一个抛出错误或者`reject`当前的Promise时，就会网Promises链上查找最近的一个`catch()`声明，执行里面的回调函数。
```
//抛出error
new Promise((resolve, reject) => {
  throw new Error('Error')
}).catch(err => {
  console.error(err)
})

// 或reject当前Promise
new Promise((resolve, reject) => {
  reject('Error')
}).catch(err => {
  console.error(err)
})

```
### 嵌套errors
在catch里面抛出异常，可以再用catch捕获异常：
```
new Promise((resolve, reject) => {
  throw new Error('Error')
})
  .catch(err => {
    throw new Error('Error')
  })
  .catch(err => {
    console.error(err)
  })
```

### promise.all 和 promise.race
#### `promises.all`
如果需要同步执行多个Promises，可以用`Promise.all()`，它可以让你定义多个Promises，并且会等所有Promises都resolved才进行后续的程序，例子：
```
const f1 = fetch('/something.json')
const f2 = fetch('/something2.json')

Promise.all([f1, f2])
  .then(res => {
    console.log('Array of results', res)
  })
  .catch(err => {
    console.error(err)
  })
```
ES2015的解构赋值还可以让你分别拿到每个Promise的返回值：
```
Promise.all([f1, f2]).then(([res1, res2]) => {
  console.log('Results', res1, res2)
})
```
#### `Promise.race()`
`Promise.race`的回调只执行一次，也就是需要竞争，最先resolve或reject的Promise会执行回调，之后就不再执行了：
```
const first = new Promise((resolve, reject) => {
  setTimeout(resolve, 500, 'first')
})
const second = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, 'second')
})

Promise.race([first, second]).then(result => {
  console.log(result) // 打出second，因为第二个等待时间更少，先返回
})
```

### 常见的坑
#### `Uncaught TypeError: undefined is not a promise`
检查一下是不是没有new, `new Promise()` 而不是`Promise()`
#### `UnhandledPromiseRejectionWarning`
这个是因为reject了一个Promise，但是在调用的时候没有用`catch`处理，加上catch就好。
