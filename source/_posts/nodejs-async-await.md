---
title: nodejs async/await
date: 2021-09-24 22:22:26
tags: ['nodejs']
---
#### 为什么引入`async/await`
`Async`函数是`promises`和`generators`的结合，基本是就是对`Promise`的更高级封装，换句话说: `async/await `是建立在`Promise`的基础上的。

`promises`在`ES2015`被引入的时候，旨在解决异步代码的问题，并且它也确实做到了，但是经过两年的发展，从`ES2015`到`ES2017`，事实证明，它并不是终极解决方案。
`promises`的引入解决了著名的`callback hell`问题，但是它本身也有一定的复杂性，包括语法上的复杂性。
应运而生的`async`函数，可以让开发者在用Promise的时候，不再固定的写一堆`catch/then`，并且避免写过长的Promise链，它让代码看起来像是同步执行，但其实背后是异步非阻塞执行。
#### 如何工作
`async`函数返回一个Promise，如：
```
const doSomethingAsync = () => {
  return new Promise(resolve => {
    setTimeout(() => resolve('I did something'), 3000)
  })
}
```
当你在调用它并且在前面加上`await`，执行的代码会停止等待，直到`promise`返回`resolved`或`rejected`状态。
注意：`await`要在`async`函数中使用。
```
const doSomething = async () => {
  console.log(await doSomethingAsync())
}
```
#### 异步执行的例子
```
const doSomethingAsync = () => {
return new Promise(resolve => {
  setTimeout(() => resolve('I did something'), 3000)
})
}
const doSomething = async () => {
  console.log(await doSomethingAsync())
}
console.log('Before')
doSomething()
console.log('After')
// 输出结果：
Before
After
I did something
```
在任意函数前面加上`async`意味着该方法会返回一个Promise, 以下例子可以看出：
```
const aFunction = async () => {
  return 'test'
}
aFunction().then((data) => console.log(data))
//相当于
const aFunction = () => {
  return Promise.resolve('test')
}
aFunction().then((data) => console.log(data))
```
#### 使代码更易读
看一下`Promise`和`async/await`的对比
```
// promise
const getFirstUserData = () => {
  return fetch('/users.json') // get users list
    .then(response => response.json()) // parse JSON
    .then(users => users[0]) // pick first user
    .then(user => fetch(`/users/${user.name}`)) // get user data
    .then(userResponse => userResponse.json()) // parse JSON
}
getFirstUserData()
//async
const getFirstUserData = async () => {
  const response = await fetch('/users.json') // get users list
  const users = await response.json() // parse JSON
  const user = users[0] // pick first user
  const userResponse = await fetch(`/users/${user.name}`) // get user data
  const userData = await userResponse.json() // parse JSON
  return userData
}
getFirstUserData()
```