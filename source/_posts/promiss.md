---
title: Promise对象入门
date: 2019-02-22 23:35
tags:
  - javascript
  - promiss
photos: 

---

### 简介
---

promise对象可以获取异步操作的消息，提供统一的API,各个异步操作都可以用同样的方法进行处理。
promise对象不受外界影响，其有三种状态：`pending`（进行中）、`fulfilled`（成功）、`rejected`（失败），只有异步操作的结果可以决定当前状态，一旦状态改变就不可以再变化，状态改变方向有两种：`pending` -> `fulfilled`、`pending` -> `rejected`  
promise对象的意义就在于将异步操作以同步操作的流程表达，避免层层嵌套的回调函数  

<!--more-->
### 基本用法

```javascript
let promise = new Promise(function (resolve, reject) {
  if () {
    resolve(value) // 异步操作成功
  } else {
    reject(error) // 失败抛错
  }
})
```

Promise构造函数接受一个函数作为参数，该函数有两个参数：`resolve`、`reject`，当执行`resolve`函数时`Promise`对象状态`pending` -> `fulfilled`，当执行reject时Promise对象状态`pending` -> `rejected`

```javascript
promise.then(function (value) {
  
}, function (error) {
  
})
```

Promise实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数，第二个参数为可选参数，例子：

```javascript
let promise = new Promise(function (resolve, reject) {
  console.log('promise')
  resolve('11')
})
promise.then(function (value) {
  console.log(value)
})
console.log('22')
```

执行结果'promise -> 22 -> 11',promise对象新建后立即执行，`then`方法的回调会在所有同步任务执行完成后执行  

### catch
`promise.prototype.catch()`是`then()`方法的别名，用于指定发生错误时的回调函数

```javascript
new Promise(function () {

}).then(() => {

}).catch(err => {
  console.log(err)
})
```

如果异步操作抛出错误，状态就会变为`reject`，就会调用`catch`中的回调，当状态为`resolve`，执行`then`方法中的回调时，若报错同样回进入`catch`的回调
意义：当我们使用promise异步操作时，但是没有使用`catch`捕获错误时，若promise异步执行报错时，外部代码并不会接收到错误，而是继续执行不受影响

```javascript
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    resolve(x);
  });
};
someAsyncThing().then(function() {
  console.log('ok');
});
setTimeout(() => { console.log('continue') }, 100);
```

如代码所示，`x`变量并没有定义，期待的操作是执行报错，然后停止运行，实际上`continue`会执行输出，这说明当没有catch捕获错误时，外部代码不会知道Promise对象内部执行已经报错，因此会继续执行。

### finally
不管Promise对象最后结果如何，都会执行的操作，`finally`方法中的回调函数不接受任何参数
```javascript
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
```
### all
`Promise.all`方法用于将多个Promise实例包装成一个新的实例
```javascript
Promise.all([p1,p2,p3]).then((array) => {
  
}).catch((err) => {
  
})
```

只有`p1`、`p2`、`p3`的状态都变成`fulfilled`，`p`的状态才会变成`fulfilled`，此时`p1`、`p2`、`p3`的返回值组成一个数组，传递给`p`的回调函数。
只要`p1`、`p2`、`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，此时第一个被`reject`的实例的返回值，会传递给`p`的回调函数。