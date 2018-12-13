---
title: JS中各类型值对应布尔值
categories:
  - JavaScript
tags:
  - JavaScript
translate_title: boolean-values-corresponding-to-various-types-of-values-in-js
date: 2018-12-06 16:30:33
---

#### Boolean 对象描述
> 在 JavaScript 中，布尔值是一种基本的数据类型。Boolean 对象是一个将布尔值打包的布尔对象。Boolean 对象主要用于提供将布尔值转换成字符串的 toString() 方法。
> 
> 当调用 toString() 方法将布尔值转换成字符串时（通常是由 JavaScript 隐式地调用），JavaScript 会内在地将这个布尔值转换成一个临时的 Boolean对象，然后调用这个对象的 toString() 方法。

#### 使用`Boolean`包装函数
``` javascript
Boolean(undefined) // false
Boolean(null) // false
Boolean(0) // false
Boolean('') // false
Boolean(NaN) // false

Boolean(1) // true
Boolean('false') // true
Boolean([]) // true
Boolean({}) // true
Boolean(function () {}) // true
Boolean(/foo/) // true
```

#### 使用双重的否运算符(`!!`)
``` javascript
!!undefined // false
!!null // false
!!0 // false
!!'' // false
!!NaN // false

!!1 // true
!!'false' // true
!![] // true
!!{} // true
!!function(){} // true
!!/foo/ // true
```

#### 注意，`对于一些特殊值，Boolean对象前面加不加new，会得到完全相反的结果，必须小心`
``` javascript
if (Boolean(false)) {
  console.log('true');
} // 无输出

if (new Boolean(false)) {
  console.log('true');
} // true

if (Boolean(null)) {
  console.log('true');
} // 无输出

if (new Boolean(null)) {
  console.log('true');
} // true
```

#### 参考文章
[Boolean 对象](http://wangdoc.com/javascript/stdlib/boolean.html)
