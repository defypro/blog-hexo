---
title: vue浏览器返回监听
tags:
  - Vue
url: 128.html
id: 128
comments: false
categories:
  - 前端
date: 2018-09-11 11:39:21
---

### 1. 挂载完成后，判断浏览器是否支持popstate
``` javascript
    mounted(){
      if (window.history && window.history.pushState) {
        history.pushState(null, null, document.URL);
        window.addEventListener('popstate', this.goBack, false);
      }
    }
``` 

### 2\. 页面销毁时，取消监听。否则其他vue路由页面也会被监听
``` javascript
    destroyed(){
      window.removeEventListener('popstate', this.goBack, false);
    }
```
    

### 3、将监听操作写在methods里面，removeEventListener取消监听内容必须跟开启监听保持一致，所以函数拿到methods里面写
``` javascript
    methods:{
      goBack(){
        this.$router.replace({path: '/'});
        //replace替换原路由，作用是避免回退死循环
      }
    }
```
