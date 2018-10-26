---
title: 发布一个属于自己的npm包
tags:
  - npm
  - javascript
categories:
  - 前端
date: 2018-10-26 11:11:11
---

发布一个属于自己的npm包

##### 注册NPM
> 首先登陆[npm官网](https://www.npmjs.com/)注册一个账号，其中`Username`就是自己的登陆账号也是用于后面命名包，例如我的包[@defy/wx-jssdk](https://www.npmjs.com/package/@defy/wx-jssdk)

##### 初始化项目
> 执行初始化命令，会让你填选一些信息，一路回车下去就好，后续可以在pkg里面改

```bash
    npm init
```

##### 修改项目结构
> 在根目录创建文件夹src，并且新建index.js
```javascript
    var hello = {
        say: function () {
            console.log('hello');
        }
    }
    module.exports = hello;
```
> 修改package.json
```json
{
  "name": "@你的username/包名",
  "version": "0.0.1",
  "main": "src/index.js",
}
```

##### 准备发布
```bash
    #执行登录，会让你输入用户名密码和邮箱，显示Logged in as xxxx on https://registry.npmjs.org/.就是成功了
    npm login

    #执行发布命令因为包名加了scope所以手动设置public
    npm publish --access=public

    #发布成功会收到邮件，也会显示包名+版本号，记住每次发布版本号必须大于上一次发布的版本号
```

##### 下载安装你的npm包
```bash
    npm install @你的username/包名 -S
```

##### 结束
