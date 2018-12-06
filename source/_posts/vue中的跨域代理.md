---
title: vue中的跨域代理
tags:
  - JavaScript
  - Vue
url: 139.html
id: 139
comments: false
categories:
  - 前端
date: 2018-10-15 09:58:57
---

vue中的跨域代理 通常我们使用`npm run dev`运行的是在`localhost:8080`下面运行的,
如果api接口没有对跨域进行设置的话那么此时浏览器同源策略会告诉你无法请求接口
在vue-cli2.0创建的项目中我们可以在`config文件夹下的index.js配置文件中`对`proxyTable`进行如下配置

    ``` javascript
    dev: {
        proxyTable: {
    		'/api': {    //将www.exaple.com印射为/apis
                target: 'https://www.exaple.com',  // 接口域名
                changeOrigin: true,  //是否跨域         
            }
    	}
      }
    ```

在vue-cli3.0创建的项目中我们可以在根目录创建vue.config.js进行如下配置

    ``` javascript
    module.exports = {
      devServer: {
        proxy: {
          '/api': {
            target: 'https://www.exaple.com',
            changeOrigin: true
          }
        }
      }
    }
    ```

那么这个时候使用axios或者其他插件进行ajax请求时，只需要请求`localhost:8080/api`就会自动转发到`https://www.exaple.com`
