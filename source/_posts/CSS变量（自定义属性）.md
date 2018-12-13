---
title: CSS变量（自定义属性）
tags:
  - CSS
url: 57.html
id: 57
comments: false
categories:
  - 前端
translate_title: css-variables-(custom-properties)
date: 2018-09-03 14:37:33
---

我们知道使用[sass](https://www.sass.hk)和[less](http://lesscss.cn)这样的预处理器可以让我们在css中使用变量等便利的特性。 
随着css新特性的不断增加，官方也提供了css变量供我们使用，叫做自定义属性，下面我们来看看怎么使用它。
### 变量的声明
声明变量的时候，变量名前面要加两根连词线（--）
```css
    :root {
      --foo: #EEEEEE;
      --bar: #FFFFFF;
    }
```  

这样我们就在根节点声明了两个自定义的变量`foo`和`bar`
```css
    :root {
      --Foo: #73A4F4;
      --foo: #EEEEEE;
    }
```      

需要注意的是变量名的大小写是敏感的`Foo`和`foo`是两个不同的变量

### 变量的使用

使用`var()`函数读取变量
```css
    a {
      color: var(--foo);
      text-decoration-color: var(--bar);
    }
```   

`var()`函数还可以使用第二个参数，表示变量的默认值。如果该变量不存在，就会使用这个默认值。
```css
    a {
      color: var(--foo, #7F583F);
    }
```     

第二个参数不处理内部的逗号或空格，都视作参数的一部分。
```css
    var(--font-stack, "Roboto", "Helvetica");
    var(--pad, 10px 15px 20px);
```       

var()函数还可以用在变量的声明。
```css
    :root {
      --primary-color: red;
      --logo-text: var(--primary-color);
    }
```     

### 兼容性处理

对于不支持 CSS 变量的浏览器，可以采用下面的写法。
```css
    a {
      color: #7F583F;
      color: var(--primary);
    }
```     

也可以使用@support命令进行检测。
```css
    @supports ( (--a: 0)) {
      /* supported */
    }
    
    @supports ( not (--a: 0)) {
      /* not supported */
    }
```     

### 参考链接

[CSS 变量教程](http://www.ruanyifeng.com/blog/2017/05/css-variables.html "CSS 变量教程") 
[Winning with CSS Variables](https://vgpena.github.io/winning-with-css-variables/ "Winning with CSS Variables")
