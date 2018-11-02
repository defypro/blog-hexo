---
title: 搭建自己的ngrok内网穿透服务
date: 2018-11-02 17:06:53
categories:
  - Linux
tags: 
    - Ngrok 
    - Linux
---

### 为什么需要内网穿透
很实用的一个场景，我们在进行微信公众号开发的时候，进行用户授权、调用JSDDK、处理公众号消息的时候需要一个可以访问的域名，如果每次都往服务器上传很麻烦，为了解决问题我们可以使用内网转发工具。
比如国外的[https://ngrok.com/](https://ngrok.com/)，国内的[https://ngrok.cc/](https://ngrok.cc/)、[echosite](https://echosite.2bdata.com/)、[https://natapp.cn](https://natapp.cn)

### 搭建自己的内网穿透
上面的这些使用起来有诸多限制，例如：不稳定，收费，不能自定义域名等等，所以决定自己用ngrok撸一个。
首先准备一个域名，将域名解析到自己的服务器，在域名解析中添加两条A记录
`your-domain.com`和`*.ngrok.your-domain.com`

### 开始安装

#### GO安装
```bash
#下载 Go 语言文件
#64位
wget https://www.golangtc.com/static/go/1.9.2/go1.9.2.linux-amd64.tar.gz
#32位
wget https://www.golangtc.com/static/go/1.9.2/go1.9.2.linux-386.tar.gz

#解压
tar -xzf go1.9.2.linux-amd64.tar.gz -C /usr/local

#增加环境变量
vim /etc/profile
export PATH=$PATH:/usr/local/go/bin

#测试
go version
```



#### 安装ngrok
```bash
#下载ngrok源码
git clone https://github.com/inconshreveable/ngrok.git ngrok
cd ngrok

#生成证书
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=ngrok.your-domain.com" -days 5000 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=ngrok.your-domain.com" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000

#替换证书
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp device.crt assets/server/tls/snakeoil.crt
cp device.key assets/server/tls/snakeoil.key

#生成ngrok的服务端和客户端，32位的GOARCH等于386，64位的是amd64自行替换，mac段的GOOS是darwin

#linux服务端
GOOS=linux GOARCH=amd64 make release-server
#win客户端
GOOS=windows GOARCH=amd64 make release-client

#现在进入bin目录，ngrokd就是服务端，windows_amd64里面的ngrok就是客户端

#后台启动服务端ngrok，这里手动设置访问端口，默认使用80和443端口
nohup bin/ngrokd -domain="ngrok.your-domain.com" -httpAddr=":8777" -httpsAddr=":8778"   > /dev/null 2>&1 &

#将客户端ngrok.exe下载到windows电脑上，在ngrok.exe的同级目录新建`ngrok.cfg`文件输入以下内容
server_addr: "ngrok.your-domain.com:4443"
trust_host_root_certs: false

#然后启动客户端
./ngrok -subdomain example -config=ngrok.cfg 80
#这样你就将example.ngrok.your-domain.com:8777映射到了本地的80端口
```

### 设置nginx反向代理
上面我们已经成功搭建好了ngrok，但是在访问的时候得加上端口号，所以我们使用nginx进行反向代理，新建nginx配置文件，输入一下内容

```bash
upstream ngrok {
        server 127.0.0.1:8777; # 此处端口要跟 启动服务端ngrok 时指定的端口一致
        keepalive 64;
}
server {
        listen 80;
        server_name *.ngrok.your-domain.com;
        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host  $http_host:8777; # 此处端口要跟 启动服务端ngrok 时指定的端口一致
                proxy_set_header X-Nginx-Proxy true;
                proxy_set_header Connection "";
                proxy_pass      http://ngrok;
        }
}
```
重启nginx
```bash
service nginx reload
```
这个时候我们就只需要访问example.ngrok.your-domain.com就可以了，nginx会自动帮我们转发到example.ngrok.your-domain.com:8777上

### 参考文章
[https://blog.csdn.net/truong/article/details/73250683](https://blog.csdn.net/truong/article/details/73250683)
[http://aevitx.com/2016/03/31/ngrok/#%E7%BC%96%E8%AF%91linux%E7%AB%AF%E7%89%88%E6%9C%AC](http://aevitx.com/2016/03/31/ngrok/#%E7%BC%96%E8%AF%91linux%E7%AB%AF%E7%89%88%E6%9C%AC)

