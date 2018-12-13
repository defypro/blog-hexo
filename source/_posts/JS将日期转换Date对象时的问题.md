---
title: JS将日期转换Date对象时的问题
tags:
  - 前端
  - JavaScript
url: 132.html
id: 132
comments: false
categories:
  - JavaScript
translate_title: problems-with-js-converting-date-objects
date: 2018-09-17 17:12:56
---

二话不说上代码 在ES5之中，如果日期采用连词线（-）格式分隔，且具有前导0，JavaScript会认为这是一个ISO格式的日期字符串，导致返回的时间是以UTC时区计算的

``` javascript
    new Date('2014-01-01')
    // Wed Jan 01 2014 08:00:00 GMT+0800 (CST)
    
    new Date('2014-1-1')
    // Wed Jan 01 2014 00:00:00 GMT+0800 (CST)
```   

上面代码中，日期字符串有没有前导0，返回的结果是不一样的。如果没有前导0，JavaScript引擎假设用户处于本地时区，所以本例返回0点0分。如果有前导0（即如果你以ISO格式表示日期），就假设用户处于格林尼治国际标准时的时区，所以返回8点0分。但是，ES6改变了这种做法，规定凡是没有指定时区的日期字符串，一律认定用户处于本地时区。 问题解决
``` javascript
    new Date('2014/01/01')
    // Wed Jan 01 2014 00:00:00 GMT+0800 (CST)
    
    new Date('2014/1/1')
    // Wed Jan 01 2014 00:00:00 GMT+0800 (CST)
```
