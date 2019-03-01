---
title: 理解14道Javascript Quiz 
categories:  javascript
tags: 
- test
---


```javascript
(function(){
    return typeof arguments;
})();
```
答案： "object"
> arguments 是对象，虽然像数组.就算是数组，typeof 返回的也是 "object" 

<br/>
<br/>

```javascript
var f = function g(){ return 23; };
typeof g();
```
答案：Error.
>  g 未定义。在 JS 里，声明函数只有 2 种方法：第 1 种： function foo(){...}  （函数声明）第 2 种： var foo = function(){...}   （等号后面必须是匿名函数，这句实质是函数表达式）除此之外，类似于 var foo = function bar(){...} 这样的东西统一按 2 方法处理，即在函数外部无法通过 bar 访问到函数，因为这已经变成了一个表达式。此处g是函数名,然而第一行不是一个函数声明,因此函数名g仅能在该函数内部被访问到

<br/>
<br/>

```javascript
(function(x){
    delete x;
    return x;
})(1);
```
答案：1
> delete 操作符用于删除对象的成员变量，不能删除函数的参数及变量。

<br/>
<br/>


```javascript
  var y = 1, x = y = typeof x;
  x;
```
答案："undefined"
> 先定义了 y 并赋值为 1，然后将 typeof x 赋值给 y ，此时 x 未定义，故为 "undefined"，最后将 y 的值赋给 x

<br/>
<br/>

```javascript
(function f(f){
    return typeof f();
})(function(){ return 1; });
```
答案："number"
> 在函数里的 f() 其实是参数的那个 f 的执行结果，所以是 typeof 1，也就是 "number"

<br/>
<br/>

```javascript
var foo = {
    bar: function() { return this.baz; },
    baz: 1
  };
  (function(){
    return typeof arguments[0]();
  })(foo.bar);
```
答案："undefined"
> 这里的 this 指的是 arguments

<br/>
<br/>

```javascript
var foo = {
    bar: function(){ return this.baz; },
    baz: 1
  }
  typeof (f = foo.bar)();
```
答案："undefined"
> 因为CallExpression是不带有上下文信息，this会指向global；当你以foo.bar() 调用时，被调用的function是「MemberExpression」，而如果进行了f=foo.bar()赋值之后，那么function就会变成「CallExpression」了，因此this绑定就失效了

<br/>
<br/>

```javascript
 var f = (function f(){ return "1"; }, function g(){ return 2; })();
  typeof f;
```
答案："number"
> 只有最后面的函数会被执行。逗号操作符多用于声明多个变量，逗号操作符还可以用于赋值，在用于赋值时，逗号操作符总会返回表达式的最后一项，例如： var num = (5,1,4,8,0); //num的值为0

<br/>
<br/>

```javascript
  var x = 1;
  if (function f(){}) {
    x += typeof f;
  }
  x;
```
答案："1undefined"
> 括号内的 function f(){} 不是函数声明，会被转换成 true ，因此 f 未定义。if语句的判断部分内是一个函数f,转化为布尔值为真。(只有""(空字符串)、0、NaN、null、undefined的布尔值是false).这段代码里的function f(){}并不是函数声明。而是函数表达式。因此全局内并无f这个标识符，代码相当于  
```javascript
var x = 1;
if(true) {
    x=1+'undefined';
}
x;
```

<br/>
<br/>

```javascript
 var x = [typeof x, typeof y][1];
  typeof typeof x;
```
答案："string"
> 第一行执行完后 x === "undefined" ，所以连续求 2 次 typeof 还是 "string"


<br/>
<br/>


```javascript
  (function(foo){
    return typeof foo.bar;
  })({ foo: { bar: 1 } });
```
答案："undefined"
> typeof foo.bar 中的 foo 是参数

<br/>
<br/>



```javascript
(function f(){
    function f(){ return 1; }
    return f();
    function f(){ return 2; }
  })();
```
答案：2
> 由于声明提前，后面的 f() 会覆盖前面的 f()

<br/>
<br/>


```javascript
function f(){ return f; }
new f() instanceof f;
```
答案：false
> 构造函数不需要显式声明返回值，默认返回this值。当显式声明了返回值时，如果返回值是非对象（数字、字符串等），这个返回值会被忽略，继续返回this值。但是如果返回值是对象，那么这个显式返回值会被返回。因为 f() 内部返回了自己，故此时 new f() 的结果和 f 相等。

<br/>
<br/>

```javascript
with (function(x, undefined){}) length;
```
答案：2
> with 限定了作用域是这个函数，而一个函数对象的length属性是该函数的形参个数，function.length 返回函数的参数个数，所以是 2。undefined 虽然是关键词，但可以被覆写。但 null 不能。