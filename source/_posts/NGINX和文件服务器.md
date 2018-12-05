---
title: NGINX和文件服务器
excerpt: 本篇博客的目的是记录使用Nginx搭建文件服务器的过程。
date: 2018-12-05 11:13:27
tags: PROGRAMING/LINUX
category: Tec
---


[参考这一篇简书](https://www.jianshu.com/p/95602720e7c8)
## 前言
本篇博客的目的是记录使用Nginx搭建文件服务器的过程。
主要分为三个步骤：
1. 安装Nginx
2. 配置Nginx
3. 使用和测试
	
## 安装Nginx
[这里是官方的安装指导](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)然而好像并没有Mac os下的安装教程
[使用brew安装nginx](https://www.jianshu.com/p/6c7cb820a020)
使用`brew install nginx`来安装nginx。
![](/images/Nginx与文件服务器1.png)
## 配置Nginx
查看nginx版本

```
nginx -v
```

关闭nginx服务

```
sudo brew services stop nginx
```

重新加载nginx

```
nginx -s reload
```

停止nginx

 ```
 nginx -s stop
 ```
 

> 上述只是一些简单的操作，实际配置为文件服务器，我们还需要配置文件conf  
> [这里是一个完善的配置修改和测试](http://www.cnblogs.com/toSeek/p/6183250.html)  

为了测试方便，我们先随便在一个目录下新建一个文件夹来装文件，让nginx来访问这个目录。
我的文件目录是：**/Users/MRWhite/FileServer**
OK 开始配置：
**vim /usr/local/etc/nginx/nginx.conf**

在http标签里面加入了三句：

```
http {
    autoindex on; // show list
    autoindex_exact_size on; // show file size
    autoindex_localtime on; // show file time
}
```

然后把server里面修改为类似下面：

```
    server {
        listen       8080;
        server_name  localhost;
        root	     /Users/MRWhite/FileServer/; # add file root
        location / {
            # root   html;
            # index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

## 测试
然后重启刷新一下吧！
`termital: nginx -s reload`
这时候打开localhost:8080就可以看到文件结构：

![](/images/Nginx与文件服务器2.png)