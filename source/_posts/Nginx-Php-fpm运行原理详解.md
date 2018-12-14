---
title: Nginx+Php-fpm运行原理详解
translate_title: detailed-operating-principle-of-nginx+php-fpm
date: 2018-12-13 13:56:57
tags:
  - PHP
  - Nginx
  - PHP-FPM
categories:
  - PHP
---

<!-- #### 代理与反向代理

##### 正向代理：访问google.com
![img](http://wx2.sinaimg.cn/mw690/0060lm7Tly1fy534n203wj30fq02k749.jpg) -->

### Nginx是什么？

> Nginx (engine x) 是一个高性能的HTTP和反向代理服务，也是一个IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。
其将源代码以类BSD许可证的形式发布，因它的稳定性、丰富的功能集、示例配置文件和低系统资源的消耗而闻名。2011年6月1日，nginx 1.0.4发布。
Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，并在一个BSD-like 协议下发行。其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。来自百度百科

### PHP-FPM是什么？

了解PHP-FPM前我们需要先知道`cgi`、`php-cgi`、`fast-cgi`

#### CGI

> 为了解决不同的语言解释器(如php、python解释器)与webserver的通信，于是出现了cgi协议。只要你按照cgi协议去编写程序，就能实现语言解释器与webwerver的通信。如php-cgi程序。

> CGI 是Web 服务器运行时外部程序的规范,按CGI 编写的程序可以扩展服务器功能。CGI 应用程序能与浏览器进行交互,还可通过数据库API 与数据库服务器等外部数据源进行通信,从数据库服务器中获取数据。格式化为HTML文档后，发送给浏览器，也可以将从浏览器获得的数据放到数据库中。几乎所有服务器都支持CGI,可用任何语言编写CGI,包括流行的C、C ++、VB 和Delphi 等。CGI 分为标准CGI 和间接CGI两种。标准CGI 使用命令行参数或环境变量表示服务器的详细请求，服务器与浏览器通信采用标准输入输出方式。间接CGI 又称缓冲CGI,在CGI 程序和CGI 接口之间插入一个缓冲程序，缓冲程序与CGI 接口间用标准输入输出进行通信。来自百度百科

#### FastCGI

有了cgi协议，解决了php解释器与webserver通信的问题，webserver终于可以处理动态语言了。但是，webserver每收到一个请求，都会去fork一个cgi进程，请求结束再kill掉这个进程。这样有10000个请求，就需要fork、kill php-cgi进程10000次。于是，出现了cgi的改良版本，fast-cgi。

> fast-cgi 是cgi的升级版本，FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一 次。PHP使用PHP-FPM(FastCGI Process Manager)，全称PHP FastCGI进程管理器进行管理。

##### FastCGI的工作原理

>1、Web Server启动时载入FastCGI进程管理器(IIS ISAPI或Apache Module)
>
>2、FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
>
>3、当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
>
>4、 FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。
>
>在上述情况中，你可以想象CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启>动时发生一次。一个额外的 好处是，持续数据库连接(Persistent database connection)可以工作。

### PHP-CGI

PHP-CGI是早期php官方出品的FastCGI管理器，也是用来解释PHP脚本的

### PHP-FPM是什么
据说是修改了php.ini配置文件后，没办法平滑重启，所以就诞生了php-fpm

PHP-FPM(FastCGI Process Manager：FastCGI进程管理器)是一个实现了Fastcgi的程序，并且提供进程管理的功能。进程包括master进程和worker进程。master进程只有一个，负责监听端口，接受来自web server的请求。worker进程一般会有多个，每个进程中会嵌入一个PHP解析器，进程PHP代码的处理。

>PHP-FPM(FastCGI Process Manager：FastCGI进程管理器)是一个PHPFastCGI管理器，对于PHP 5.3.3之前的php来说，是一个补丁包 [1]  ，旨在将FastCGI进程管理整合进PHP包中。如果你使用的是PHP5.3.3之前的PHP的话，就必须将它patch到你的PHP源代码中，在编译安装PHP后才可以使用。
相对Spawn-FCGI，PHP-FPM在CPU和内存方面的控制都更胜一筹，而且前者很容易崩溃，必须用crontab进行监控，而PHP-FPM则没有这种烦恼。来自百度百科

### Nginx与Php-fpm结合

我们可以看到NGINX的虚拟主机配置文件

``` bash
server {
    listen       80; #监听80端口，接收http请求
    server_name  www.example.com; #就是网站地址
    root /usr/local/etc/nginx/www/huxintong_admin; # 准备存放代码工程的路径
    #路由到网站根目录www.example.com时候的处理
    location / {
        index index.php; #跳转到www.example.com/index.php
    }   

    #当请求网站下php文件的时候，反向代理到php-fpm
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;#表示nginx通过fastcgi协议将用户请求的资源发给127.0.0.1:9000进行解析，这里的nginx和php脚本解析服务器是在同一台机器上，所以127.0.0.1:9000表示的就是本地的php脚本解析服务器。
        include        fastcgi_params;#加载fastcgi参数，可以理解为nginx会通过fastcgi协议处理本次请求
    }
}
```

当我们访问`www.example.com`的时候，处理流程是这样的：

```bash
  www.example.com
        |
        |
      Nginx
        |
        |
路由到www.example.com/index.php
        |
        |
加载FastCgi模块
        |
        |
Nginx通过FastCgi协议将请求转发到127.0.0.1:9000
        |
        |
PHP-FPM监听127.0.0.1:9000
        |
        |
PHP-FPM接收到请求，分配进程处理PHP
        |
        |
处理完成后返回结果NGINX
        |
        |
Nginx通过http协议响应给客户端
```

### 总结
当`Web Server`收到 `index.php` 这个请求后，会启动对应的 CGI 程序，这里就是PHP的解析器。接下来PHP解析器会解析`php.ini`文件，初始化执行环境，然后处理请求，再以规定CGI规定的格式返回处理后的结果，退出进程，Web server再把结果返回给浏览器。这就是一个完整的动态PHP Web访问流程，接下来再引出这些概念，就好理解多了，

`CGI`：是 Web Server 与 Web Application 之间数据交换的一种协议。

`FastCGI`：同 CGI，是一种通信协议，但比 CGI 在效率上做了一些优化。同样，SCGI 协议与 FastCGI 类似。

`PHP-CGI`：是 PHP （Web Application）对 Web Server 提供的 CGI
协议的接口程序。

`PHP-FPM`：是 PHP（Web Application）对 Web Server 提供的 FastCGI 协议的接口程序，额外还提供了相对智能一些任务管理。

