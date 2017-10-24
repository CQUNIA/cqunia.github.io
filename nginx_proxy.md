---
title: 同一Nginx服务器下不同服务绑定不同域名的方法
date: 2017-03-29 20:22:30
tags: [Linux,NginX,服务器]
---
&emsp;&emsp;照常来说，一台主机的IP地址只能解析到一个域名的A记录中，如果我们的服务器上有多个网络服务，或者是我们需要搭建多个网站，想把这些服务或者网站分别绑定到一个域名的不同的子域名下，如[dev.maxoyed.com](http://dev.maxoyed.com)和[maxoeyd.com](http://maxoeyd.com)分别是运行在同一台服务器上的两个网站。本文就将分享一台主机不同服务绑定不同子域名的方法。

------

>本文所测试的环境：腾讯云CentOS7.2 64位，ssh工具为XShell,http服务器为Nginx，以root用户身份操作。

## 1.修改Nginx配置文件
&emsp;&emsp;废话少说，上代码
```shell
$ vi /usr/local/nginx/conf/nginx.conf

```
&emsp;&emsp;按`Insert`键切换到输入模式，在nginx.conf的http字段的`fastcgi_busy_buffers_size 128k`和`fastcgi_temp_file_write_size 256k`之间添加`fastcgi_intercept_errors on`，输入完成后按`Esc`退出输入模式，输入`:wq`回车保存并退出。

## 2.添加vhost配置文件
&emsp;&emsp;在`/usr/local/nginx/conf/vhost`目录下添加第二个网站的配置文件，我这里取名为`dev.conf`
```shell
$ vi /usr/local/nginx/conf/vhost/dev.conf
```
&emsp;&emsp;在`dev.conf`文件中输入
```shell
server {
    listen       80;
    server_name  dev.maxoyed.com;       //填写你要绑定的域名
    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-Ip $remote_addr;
        proxy_set_header X-NginX-Proxy true;
        proxy_pass http://127.0.0.1:8081/;      //你的网站服务端口，自己更改
        proxy_redirect off;
    }
}
```
&emsp;&emsp;保存并退出，输入`/etc/init.d/nginx restart`重启Nginx服务器，在浏览器地址栏中输入[dev.maxoyed.com](http://dev.maxoyed.com)，可以正确访问。


