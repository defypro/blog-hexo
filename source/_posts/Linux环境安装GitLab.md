---
title: Linux环境安装GitLab
tags:
  - GitLab
url: 124.html
id: 124
comments: false
categories:
  - Linux
translate_title: installation-of-gitlab-in-linux-environment
date: 2018-09-05 16:16:14
---

### 安装依赖
```bash
    #配置系统防火墙,把HTTP和SSH端口开放.
    sudo yum install curl openssh-server postfix cronie
    sudo service postfix start
    sudo lokkit -s http -s ssh
    sudo chkconfig postfix on
```

### 下载安装GitLab包

GitLab下载地址:https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6
```bash
    wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6/gitlab-ce-11.1.6-ce.0.el6.x86_64.rpm
    rpm -Uvh gitlab-ce-11.1.6-ce.0.el6.x86_64.rpm
```

### 修改gitlab配置文件指定服务器ip和自定义端口

```bash
    vim /etc/gitlab/gitlab.rb
    #禁用gitlab附带的nginx
    nginx['enable'] = false
    gitlab_workhorse['enable'] = true
    gitlab_workhorse['listen_network'] = "tcp"
    #设置gitlab监听端口
    gitlab_workhorse['listen_addr'] = "127.0.0.1:8181"
    
    #配置邮件
    gitlab_rails['gitlab_email_from'] = 'xxxxx@qq.com'
    
    gitlab_rails['smtp_enable'] = true
    gitlab_rails['smtp_address'] = "smtp.qq.com"
    gitlab_rails['smtp_port'] = 465
    gitlab_rails['smtp_user_name'] = "xxxxx@qq.com"
    gitlab_rails['smtp_password'] = "xxxxx"
    gitlab_rails['smtp_domain'] = "smtp.qq.com"
    gitlab_rails['smtp_authentication'] = "login"
    gitlab_rails['smtp_enable_starttls_auto'] = true
    gitlab_rails['smtp_tls'] = true
```
    

### 创建nginx配置文件
```bash
    vim /etc/nginx/conf.d/gitlab.xxx.com.conf
    
    upstream gitlab-workhorse {
      server 127.0.0.1:8181; #根据实际情况修改
    }
    server {
      listen 80;
      listen [::]:80 default_server;
      server_name gitlab.xxx.com; ## 修改成自己的域名；
      server_tokens off; ## Don't show the nginx version number, a security best practice
      root /opt/gitlab/embedded/service/gitlab-rails/public;
    
      ## See app/controllers/application_controller.rb for headers set
    
      ## Individual nginx logs for this GitLab vhost
      ## access_log  /home/wwwlogs/gitlab_access.log; # 根据实际情况修改
      ## error_log   /home/wwwlogs/gitlab_error.log; # 根据实际情况修改
    
      location / {
        client_max_body_size 0;
        gzip off;
    
        ## https://github.com/gitlabhq/gitlabhq/issues/694
        ## Some requests take more than 30 seconds.
        proxy_read_timeout      300;
        proxy_connect_timeout   300;
        proxy_redirect          off;
    
        proxy_http_version 1.1;
    
        proxy_set_header    Host                $http_host;
        proxy_set_header    X-Real-IP           $remote_addr;
        proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto   $scheme;
    
        proxy_pass http://gitlab-workhorse;
      }
    }
```

### 重载配置
```bash
    #重载配置
    sudo gitlab-ctl reconfigure
    #启动
    gitlab-ctl start
    #停止
    gitlab-ctl stop
    #重启
    gitlab-ctl restart
```

打开浏览器访问gitlab.xxx.com
