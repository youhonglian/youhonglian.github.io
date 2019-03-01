---
title: Javascript Test
categories:  javascript
tags: 
- test
---



### 字符串反转，输入一串字符串，包括逗号和句号(, .)，整体反转但单词不反转，包括空格，实现以下效果 
 ```javascript 
 输入 '   hello,world...   '  
 输出 '    ...world,   hello'
```
<br/>

参考：
```javascript
function test(str) {
    var arr = []; 
    for(var i = 0; i<str.split(',').length; i++) {
        var subStr = str.split(',')[i];
        var length = subStr.lastIndexOf('.')-subStr.indexOf('.')
        if(length>0) {
            var first = subStr.indexOf('.')
            var last = subStr.lastIndexOf('.')
            subStr = subStr.slice(last+1)+ subStr.slice(first, first+length+1) + subStr.slice(0, first)
            arr.push(subStr)
        } else {
            arr.push(subStr);
        }
    }
    var reStr =arr.reverse().join(',')
    retun restr
}
test('   hello,world...   ')
```
<br/>
<br/>

### 为了得到一个数的"相反数",我们将这个数的数字顺序颠倒,然后再加上原先的数得到"相反数"。例如,为了得到1325的"相反数",首先我们将该数的数字顺序颠倒,我们得到5231,之后再加上原先的数,我们得到5231+1325=6556.如果颠倒之后的数字有前缀零,前缀零将会被忽略。例如n = 100, 颠倒之后是1. 

 ```javascript 
输入例子1:
1325
输出例子1:
6556
输入例子2:
100
输出例子2:
101
```
<br/>

参考： 
```javascript
   function test(p) {
        if(1 <= p <= 10^5) {
            return parseInt(p.toString().split('').reverse().join('')) + p   
        }
    }
    test(100)
```