---
title: 浅谈Promise
categories:  ES6
tags: 
- JS
---

## Promise 简介
> Promise 对象是用来处理异步操作的工具,解决开发者对异步回调的烦恼。可以说Promise是个代理对象，在设计模式来讲就是代理模式，它代理了一个值（通过resolve方法传递的值），并且设置了几个状态让用户知道当前代理值解析的结果。

## Promise 状态

按照规范，Promise有三种状态：

* pending: 初始状态,未完成或拒绝，可改变状态。

* fulfilled（resolved）: 操作成功完成,不可改变状态,拥有不可变的终值。

* rejected: 操作失败,不可改变状态,拥有不可变的拒因。

## Promise 术语

* 解决（fulfill）：指一个 promise 成功时进行的一系列操作，如状态的改变、回调的执行。虽然规范中用 fulfill 来表示解决，但在后世的 promise 实现多以 resolve 来指代之。

* 拒绝（reject）：指一个 promise 失败时进行的一系列操作。

* 终值（eventual value）：所谓终值，指的是 promise 被解决时传递给解决回调的值，由于 promise 有一次性的特征，因此当这个值被传递时，标志着 promise 等待态的结束，故称之终值，有时也直接简称为值（value）。

* 据因（reason）：也就是拒绝原因，指在 promise 被拒绝时传递给拒绝回调的值。

## 状态机制切换

状态只能由pengding-->fulfilled，或者由pending-->rejected这样转变。

只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。就算改变已经发生了，你再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

## 基本API
>Promise.then()  <br/>
语法：Promise.prototype.then(onFulfilled, onRejected)
对promise添加onFulfilled和onRejected回调，并返回的是一个新的Promise实例（不是原来那个Promise实例），且返回值将作为参数传入这个新Promise的resolve函数。
>.catch() <br/>
语法：Promise.prototype.catch(onRejected)
该方法是.then(undefined, onRejected)的别名，用于指定发生错误时的回调函数。
>.all() <br/>
语法：Promise.all(iterable)
该方法用于将多个Promise实例，包装成一个新的Promise实例。

var p = Promise.all([p1, p2, p3]);
Promise.all方法接受一个数组（或具有Iterator接口）作参数，数组中的对象（p1、p2、p3）均为promise实例（如果不是一个promise，该项会被用Promise.resolve转换为一个promise)。它的状态由这三个promise实例决定。
>.race()<br/>
语法：Promise.race(iterable)
该方法同样是将多个Promise实例，包装成一个新的Promise实例。
例如：var p = Promise.race([p1, p2, p3]);
Promise.race方法同样接受一个数组（或具有Iterator接口）作参数。当p1, p2, p3中有一个实例的状态发生改变（变为fulfilled或rejected），p的状态就跟着改变。并把第一个改变状态的promise的返回值，传给p的回调函数。

## Promise常见问题
* 在异步回调中抛错，不会被catch到
* promise状态变为resove或reject，就凝固了，不会再改变
* 如果在then中抛错，而没有对错误进行处理（即catch），那么会一直保持reject状态，直到catch了错误
* reject 和 catch 的区别:
promise.then(onFulfilled, onRejected) //在onFulfilled中发生异常的话，在onRejected中是捕获不到这个异常的。
promise.then(onFulfilled).catch(onRejected)  //.then中产生的异常能在.catch中捕获


## 手写一个Promise

```javascript
function Promise(callback) {
  var self = this
  self.status = 'pending' // Promise当前的状态
  self.data = undefined  // Promise的值
  self.onResolvedCallback = [] // Promise resolve时的回调函数集
  self.onRejectedCallback = [] // Promise reject时的回调函数集
  callback(resolve, reject) // 执行callback并传入相应的参数

  function resolve(value){
      if(self.status=='pending'){
          self.status=='resolved';
          self.data=value;
          // 依次执行成功之后的函数栈
          for(var i = 0; i < self.onResolvedCallback.length; i++) {
            self.onResolvedCallback[i](value)
          }
      }
  }

  function rejecet(error){
    if (self.status === 'pending') {
       self.status = 'rejected'
       self.data = error;
       // 依次执行失败之后的函数栈
       for(var i = 0; i < self.onRejectedCallback.length; i++) {
           self.onRejectedCallback[i](error)
        }
    }
  }
}

```

<br/>

接下来我们实现我们的then方法, then方法是Promise的核心, 一个promise的then接受两个参数,首先需要判断参数是否为函数，若不为函数忽略

```javascript
Promise.prototype.then = function(onResolved, onRejected) {
  var self = this
  var promise2

  onResolved = typeof onResolved === 'function' ? onResolved : function(value) {}
  onRejected = typeof onRejected === 'function' ? onRejected : function(reason) {}

  if (self.status === 'resolved') {
    return promise2 = new Promise(function(resolve, reject) {
      try {
        var x = onResolved(self.data)
        if (x instanceof Promise) { 
          x.then(resolve, reject)
        }
        resolve(x) // 否则，以它的返回值做为promise2的结果
      } catch (e) {
        reject(e) // 如果出错，以捕获到的错误做为promise2的结果
      }
    })
  }

  // 此处与前一个if块的逻辑几乎相同，区别在于所调用的是onRejected函数，就不再做过多解释
  if (self.status === 'rejected') {
    return promise2 = new Promise(function(resolve, reject) {
      try {
        var x = onRejected(self.data)
        if (x instanceof Promise) {
          x.then(resolve, reject)
        }
      } catch (e) {
        reject(e)
      }
    })
  }
```

<br/>


如果当前的Promise还处于pending状态，我们并不能确定调用onResolved还是onRejected，
只能等到Promise的状态确定后，才能确实如何处理。
所以我们需要把我们的两种情况的处理逻辑做为callback放入promise1(此处即this/self)的回调数组里
逻辑本身跟第一个if块内的几乎一致，此处不做过多解释

<br/>

```javascript
  if (self.status === 'pending') {
    return promise2 = new Promise(function(resolve, reject) {
      self.onResolvedCallback.push(function(value) {
        try {
          var x = onResolved(self.data)
          if (x instanceof Promise) {
            x.then(resolve, reject)
          }
        } catch (e) {
          reject(e)
        }
      })

      self.onRejectedCallback.push(function(reason) {
        try {
          var x = onRejected(self.data)
          if (x instanceof Promise) {
            x.then(resolve, reject)
          }
        } catch (e) {
          reject(e)
        }
      })
    })
  }
}
```

我们顺便实现一个catch方法

```javascript
Promise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected)
}

```