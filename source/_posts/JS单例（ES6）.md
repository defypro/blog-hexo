---
title: JS单例（ES6）
tags:
  - JavaScript
  - ES6
categories:
  - 前端
translate_title: js-singleton-(es6)
date: 2018-12-12 15:57:33
---

ES6引入`class`和`constructor`来创建对象，下面我们使用ES6实现单例模式
使用`static`关键字定义静态方法
``` javascript
    class Singleton {
        constructor(name) {
            this.name = name;
            this.instance = null;
        }

        static getInstance(name) {
            if(!this.instance) {
                this.instance = new Singleton(name);
            }
            return this.instance;
        }
    }
    
    const singleton1 = Singleton.getInstance('singleton1');
    const singleton2 = Singleton.getInstance('singleton2');
    console.log(singleton1 === singleton2); //true
```  

JS中没有私有变量，所以我们借助`Symbol`
``` javascript
    const name1 = Symbol('name1');
    class Singleton {
        constructor(name) {
            this.name = name;
            this[name1] = name;
            this.instance = null;
        }

        static getInstance(name) {
            if(!this.instance) {
                this.instance = new Singleton(name);
            }
            return this.instance;
        }
    }
    
    const singleton1 = Singleton.getInstance('singleton1');
    console.log(singleton1.name); //singleton1
    console.log(singleton1.name1); //undefined
```  
