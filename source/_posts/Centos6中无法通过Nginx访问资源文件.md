---
title: Centos6中无法通过Nginx访问资源文件
tags:
  - Linux
  - Nginx
url: 36.html
id: 36
comments: false
categories:
  - Linux
translate_title: resource-files-cannot-be-accessed-through-nginx-in-centos6
date: 2018-08-31 15:07:40
---

在谷歌云搞了个免费的服务器，搭建了一个wordpress结果css、js一直报404，一开始以为是nginx的启动用户和文件用户不一样，
检查了一遍nginx.conf配置，将user nobody改成了user www，重启nginx发现还是404，
网上各种搜索于是找到了这篇文章[Nginx 因 Selinux 服务导致无法远程访问](https://blog.csdn.net/johnnycode/article/details/41947581 "Nginx 因 Selinux 服务导致无法远程访问")解决了我的问题。
下面简单说说吧，其实还是不太理解。 
发现网站有资源文件404的话首先查看一下资源文件目录 ![blob.jpg](https://i.loli.net/2018/08/30/5b878975356db.jpg) 发现权限后面都带了一个`.`那就说明这个文件是收保护的，再运行一下getenforce查看selinux的运行模式

``` bash
$ getenforce
Enforcing
```

运行模式分为三种 enforcing (强制模式)、permissive（宽容模式）、disabled（关闭） 所以此时要解决这个问题有两种方法； 1.临时关闭

``` bash
$ setenforce 0
```

2.永久关闭，修改 /etc/selinux/config 文件（如下）

``` bash
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

重启电脑查看 Selinux 状态，应该为关闭状态

``` bash
$ getenforce
Disabled
```

此时新建文件，查看文件权限后面也就没有带`.`了，静态资源也可以正常访问了
