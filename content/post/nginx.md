---
title: "Nginx服务相关"
slug: "nginx服务相关"
date: "2024-05-02"
created: "2024-05-02"
tags: ["nginx"]
---


# HTTP工作原理

# 一、HTTP介绍

### HTTP工作原理

- http协议工作于客户端-服务端架构上。浏览器作为HTTP客户端通过URL向HTTP服务端即web服务器发送所有请求。

- web服务器有：nginx、Apache服务器，IIS服务器（IInternet Information services）等。

- web根据接收的请求后，向客户端发送响应信息。

- HTTP默认端口为80

  

  http三点注意事项

- HTTP是无连接，无连接的含义是限制每次连接只处理一个请求，服务器处理完客户的请求，并收到客户的答应后，即断开连接采用这种方式可以节省传输时间。-----短连接
- TCP是长连接
- HTTP是媒体独立的：这意味着，只要客户端和服务器知道如何处理数据内容，任何类型的数据都可以通过HTTP发送，客户端以及服务器指定使用适合的MIME-type内容类型。
- HTTP是无状态： HTTP协议是无状态协议。无状态协议是指对于事务处理没有记忆能力，缺少状态，意味着如果后续处理需要前面的信息则它必须重传。





### HTTP协议通信流程

CGI Program是与后台数据库打交道，对数据库直接操作是很危险的，CGI程序可以用任何祭敖包语言或者是完全独立编程语言实现，只要这个语言可以在这个系统上运行。

Web Browser-><u>http Protocol</u>   ---->   http Serve ---->CGI Program(网关接口，后台，Java/php写的)  ---->Database      http不能直接和数据库对接

### HTTP消息结构

### 客户端请求信息 

客户端发送一个http请求到服务器的请求消息包括以下格式：以下四个部分

请求行（request line ) 

请求头部header

空行

请求数据

### 服务器响应请求

HTTP响应也由四个部分组成，状态行、消息报头，空行和响应正文   。

1. HTTP请求方法
2. HTTP响应消息
3. HTTP状态码

HTTP/1.1 200 OK

```
F12

network

ctrl+F5强制刷新，点击任意一个
headers

Request URL:
https://pss.bdstatic.com/static/superman/amd_modules/tslib-c95383af0c.js
Request Method:
GET
Status Code:  200 OK     #状态码，200代表服务正常，404页面不存在
Remote Address:
211.97.82.36:443
Referrer Policy:
unsafe-url

Response Headers指的是对方的响应，我的响应看不到，想看自己的请求，，可以装个软件jmeter，抓到自己请求，和对方的回应
Request header请求头部
```

请求一个网站

```
#客户端请求
curl -v https://www.baidu.com
Connected to www.baidu.com (153.3.238.110) port 443 (#0)
> GET / HTTP/1.1       #请求方式与版本协议，get代表客户端发起的请求
> User-Agent: curl/7.29.0   #用什么客户端访问
> Host: www.baidu.com      #主机名，域名，主机和端口
> Accept: */*        #匹配什么文件类型，“*”是通用匹配，匹配所有类型

#服务端响应：
< HTTP/1.1 200 OK    #请求返回的状态码，对方的回应
< Accept-Ranges: bytes     
< Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
< Connection: keep-alive
< Content-Length: 2443
< Content-Type: text/html
< Date: Thu, 02 May 2024 12:38:02 GMT
< Etag: "58860401-98b"
< Last-Modified: Mon, 23 Jan 2017 13:24:17 GMT
< Pragma: no-cache
< Server: bfe/1.0.8.18    #服务版本信息，这个bfe是这个公司隐藏了
< Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/

```

### HTTP请求方法

```
HTTP1.0定义了三种请求方法：
GET发起方
POST响应方
HEAD通信的头部
HTTP1.1新增了五种请求fnagfa:OPTIONS,PUT,DELLTE, TRACEhe CONNECT方法。
```

### HTTP常见的状态码

```
200  请求成功
301  资源（网页等）被永久转移其他URL
404  请求的资源（网页等）不存在，配置出问题
500  内部服务错误，后端服务挂机了
403  目录权限不允许你浏览
304  网站代码错了或者配置出问题
```

# 二、nginx进阶基础

## nginx 介绍

```
Nginx(engine x)是一个轻量级,高性能的HTTP和反向代理服务，也是一个IMAP/POP3/smtp服务.因它的稳定性,丰富的功能集\示例配置文件和低系统资源的消耗而闻名.其特点是占有内存少并发能力强,事实上nginx的并发能力在同类型的网页服务中表现较好,中国大陆使用网站用户有百度京东新浪网易腾讯淘宝等有名的版本。

第三方开发插件比较多，可做二次开发，游戏公司，互联网应用吩咐的公司自己写模块，开发语言c,c++
通信协议

开源成本低，扩容方便

在高连接并发的情况下，nginx是Apache服务器不错的替代品。

创始人伊戈尔.赛索耶夫

本地化适应性：针对中国市场特定的环境和需求，一些开源项目进行了本土化的改良，使之更符合国内企业的使用习惯和法律法规要求。您提到的“淘宝里面的Tengine”和“OpenResty”就是两个典型例子：

Tengine：这是基于Nginx开发的一个高性能Web服务器，由阿里巴巴集团发起并维护。Tengine在Nginx的基础上加入了众多企业级功能，比如动态模块加载、更强大的负载均衡支持、更细致的日志记录以及对后端服务器健康检查的增强，非常适合处理大规模、高并发的互联网应用场景，特别满足了中国电商行业的需求。

OpenResty：它是一个基于Nginx与Lua的高性能Web平台，允许开发者直接在Nginx服务器中编写Lua脚本，实现灵活的流量控制、API网关功能、安全策略等。OpenResty因其高度的灵活性和性能，在中国乃至全球范围内被广泛应用于构建高性能的Web服务和API网关，特别是在需要复杂逻辑处理和高性能请求处理的场景下。
```

## 为什么选择nginx（优势）

```
1.高并发，高性能
2.高可靠---可以24小时不间断运行
3.可扩展性强---模块化设计，使得添加模块非常的平稳
4.热部署---可以在不停止服务器的情况下升级nginx
5.BSD许可证---nginx不止开源免费，我们还可以更具实际需要进行定制修改源代码
```

nginx是一个高性能的web和反向代理服务器，它具有很多非常优越的特性
单机环境下参考服务器配置并发连接数在7000+ -8000左右   集群模式20000+

搭建

```
#yum源安装
yum -y install epel-release    #下载扩展仓库包
yum -y install yum-utils
```



## nginx编译安装

### 1.安装依赖包

```
#gcc gcc-c++ pcre pcre-devel  gd-devel()使nginx支持http rewrite模块
#openssl openssl-devel(使nginx支持ssl，证书加密组件包)
#wget 
yum -y install gcc gcc-c++ pcre pcre-devel gd-devel openssl openssl-devel zlib zlib-devel
```

### 2.创建nginx用户

```
useradd nginx
wget httP://nginx.org/download/nginx-1.26.0.tar.gz
tar xvf nginx-1.6.0.tar.gz
cd nginx-1.16.0/

#预编译
./configure  \
--prefix=/usr/local/nginx  \
--group=nginx  \
--user=nginx \
--with-http_stub_status_module \
--with-http_v2_module \
--with-http_ssl_module \
--with-http_gzip_static_module \
--with-http_realip_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-stream  \
--with-stream_ssl_module \
--with-stream_realip_module

#编译安装
make  && make install
```

#### 返回值

```
test -f '/usr/local/nginx/conf/nginx.conf' \
	|| cp conf/nginx.conf '/usr/local/nginx/conf/nginx.conf'
cp conf/nginx.conf '/usr/local/nginx/conf/nginx.conf.default'
test -d '/usr/local/nginx/logs' \
	|| mkdir -p '/usr/local/nginx/logs'
test -d '/usr/local/nginx/logs' \
	|| mkdir -p '/usr/local/nginx/logs'
test -d '/usr/local/nginx/html' \
	|| cp -R html '/usr/local/nginx'
test -d '/usr/local/nginx/logs' \
	|| mkdir -p '/usr/local/nginx/logs'
make[1]: Leaving directory `/root/nginx-1.26.0'
[root@node1 nginx-1.26.0]# echo $?
0
```

### 3.添加环境变量

```
cat >/etc/profile.d/nginx.sh<<EOF
export PATH=\${PATH}:/usr/local/nginx/sbin
EOF

source /etc/profile
```

### 4.配置文件

```
yum安装的配置文件位置：/etc/nginx/nginx.conf
源码安装配置文件：/usr/local/nginx/conf/nginx.conf

1.nginx.conf的组成：三部分，全局块，events块，http块
在http块中有包含了http全局块、多个server块。
每个server块又包含了server全局块以及多个location块。
在统一配置块中嵌套的配置块，各个之间不存在次序关系

#备份原配置文件产生的配置文件
mv /usr/local/nginx/conf/nginx.cof{,.bak}

```

### 5.编译安装的nginx.conf

```
[root@node1 conf]# cat nginx.conf

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
[root@node1 conf]# 

```

#### 检测语法

```
nginx -t

[root@node1 conf]# nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

```

#### 设置开机自启

```
cat >/usr/lib/systemd/system/nginx.service<<EOF
[Unit]
Description=nginx
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/bin/rm -f /usr/local/nginx/logs/nginx.pid
ExecStartPost=/bin/sleep 0.1
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c  /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP 
ExecStop=/bin/kill -s QUIT 
PrivateTmp=true
LimitNOFILE=51200
LimitNPROC=51200
LimitCORE=51200

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable nginx
```

#### 手动启服务

```
ss -tunlp
cd /root/nginx-1.26.0
chown -R nginx:nginx  *
ngixn -s stop
```

### 6.检查服务

```
yum install lsof -y
[root@node1 nginx-1.26.0]#  lsof -i :80
COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   30470  root    6u  IPv4  55784      0t0  TCP *:http (LISTEN)
nginx   30471 nginx    6u  IPv4  55784      0t0  TCP *:http (LISTEN)


[root@node1 nginx-1.26.0]# ss -tunlp |grep 80
tcp    LISTEN     0      128       *:80                    *:*                   users:(("nginx",pid=30471,fd=6),("nginx",pid=30470,fd=6))

```

### 7.正式开始写ngin.conf文件

```
#一个server就类似于一个主机，这样分开写，方便后期进行维护
[root@node1 conf]# pwd 
/usr/local/nginx/conf
[root@node1 conf]# 
[root@node1 conf]# cat web.conf 
server {
        listen       80;
        server_name  localhost;
        access_log  logs/server1-access.log  main;
        location / {
            root   /data/wwwroot;
            index  index.html index.htm;
        }
}
[root@node1 conf]# cat nginx.conf
#user  nobody;
worker_processes  auto;
error_log  logs/error.log;
pid        logs/nginx.pid;


events {
    worker_connections  65536;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  logs/access.log  main;
    sendfile        on;
    tcp_nopush     on;
    keepalive_timeout  65;
    gzip  on;
    include web.conf;
}
[root@node1 conf]# 
```

### 8.使用域名进行配置

```
[root@node1 ~]# mkdir -p /data/wwwroot/{www.nginx2024.com,www.nginx2023.com}
[root@node1 ~]# ll /data/wwwroot/
total 8
-rw-r--r--. 1 root root 3 May  3 02:53 check.html
-rw-r--r--. 1 root root 3 May  3 02:44 index.html
drwxr-xr-x. 2 root root 6 May  3 03:32 www.nginx2023.com
drwxr-xr-x. 2 root root 6 May  3 03:32 www.nginx2024.com
[root@node1 ~]# ll /data/wwwroot/www.nginx2023.com/
total 0
[root@node1 ~]# for i in {www.nginx2024.com,www.nginx2023.com};
> do 
> echo "site:$i"  > /data/wwwroot/$i/index.html ;
> done
[root@node1 ~]# cat /data/wwwroot/www.nginx2023.com/index.html  /data/wwwroot/www.nginx2024.com/index.html 
site:www.nginx2023.com
site:www.nginx2024.com


[root@node1 conf]# cat nginx.conf
#user  nobody;
worker_processes  auto;
error_log  logs/error.log;
pid        logs/nginx.pid;


events {
    worker_connections  65536;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  logs/access.log  main;
    sendfile        on;
    tcp_nopush     on;
    keepalive_timeout  65;
    gzip  on;
    include web.conf;
}
[root@node1 conf]# cat web.conf 
server {
        listen       80;
        server_name  www.nginx2024.com;
        access_log  logs/www.nginx2024-access.log  main;
        location / {
            root   /data/wwwroot/www.nginx2024.com;
            index  index.html index.htm;
        }
}

server {
        listen      80;
        server_name  www.nginx2023.com;
        access_log   logs/www.nginx2023.com-access.log main;
        location / {
            root   /data/wwwroot/www.nginx2023.com;
            index  index.html index.hml;
        }
}

```

### 9.域名进行访问

```
Windows系统访问配置本地域名解析C:\Windows\System32\drivers\etc
Linux配置/etc/hosts
curl -I http://www.nginx2023.com
http://www.nginx2023.com
```

# 三、nginx proxy反向代理

### 1.代理原理

```
反向代理服务端的实现：
需要一个负载均衡设备（即反向代理服务器）来分发用户请求，将用户请求分发到后端真正提供服务的服务器上。服务器返回自己的服务到负载均衡设备。负载均衡设备将服务器的服务返回用户。
```

### 2.正/反代理区别

#### 2.1 正向代理

#### 2.2反向代理

#### 2.4 区别

```
正向代理中代理的对象是客户端
反向代理中代理的是服务端
```

### 3.nginx proxy配置(代理配置)

#### 3.1代理模块

```
ngx_http_proxy_module   #默认已经安装了
```

#### 3.2启用nginx proxy代理

```
反代服务器192.168.230.173
web服务192.168.230.174
```

反向代理配置192.168.230.173

nging.conf配置文件

```
#编译安装
[root@node1 conf.d]# which nginx
/usr/local/nginx/sbin/nginx

#nginx配置文件
[root@node1 ~]# cat /usr/local/nginx/conf/nginx.conf
#user  nobody;
worker_processes  auto;
error_log  logs/error.log;
pid        logs/nginx.pid;


events {
    worker_connections  65536;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  logs/access.log  main;
    sendfile        on;
    tcp_nopush     on;
    keepalive_timeout  65;
    gzip  on;
    include conf.d/proxy.conf;
}
[root@node1 ~]#

```

#### proxy.conf配置文件

```
[root@node1 ~]# cat /usr/local/nginx/conf/conf.d/proxy.conf 
server {
       listen   80;
       server_name  oa.qf2204.com;
       #定义访问日志
       access_log  logs/proxy_oa.qf2204.com.access.log main;
       location / {
       proxy_pass http://192.168.230.174:80;
       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_connect_timeout 30;
       proxy_send_timeout 60;
       proxy_read_timeout 60;
       }
}
[root@node1 ~]# 

```



###  4.后端web服务器   192.168.230.174

#### 4.1yum安装

##### 1.配置yum源

```
[root@node2 ~]# cat /etc/yum.repos.d/nginx.repo 
[nginx-stable]
name=nginx
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1

#清楚缓存，重新生成yum
yum clean all
yum repolist

```

##### 2.安装nginx

```
yum -y install epel-release yum-utils  ngixn

#关闭防火墙selinux
systemctl stop firewalld
chown -R nginx:nginx /data/wwwroot
chmod -R 755 /data/wwwroot

#关闭selinux
vi /etc/selinux/config
SELINUX=disabled
```

##### 3.启动服务

```
systemctl start nginx
systemctl enable nginx
systemctl status  nginx
```

```
#yum源安装的显示如下
[root@node2 ~]# which nginx
/usr/sbin/nginx
```

#### 4.2配置文件nginx.conf

```
[root@node2 nginx]# cat /etc/nginx/nginx.conf
user  nginx;
worker_processes  auto;
error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    gzip  on;
    include /etc/nginx/conf.d/*.conf;
}

```

#### 4.3 配置文件 deafult.conf

```
[root@node2 nginx]# cat /etc/nginx/conf.d/default.conf
server {
    listen       80;
    server_name  oa.qf2204.com;
    access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /data/wwwroot;
        index  index.html index.htm;
    }
}

```



#### 4.4创建首页路径

```
mkdir /data/wwwroot
ll /data/wwwroot/
echo "site:oa.qf2204.com--------192.168.230.174"  >/data/wwwroot/index.html
cat /data/wwwroot/index.html 

```

#### 4.5测试

```
curl -I http://192.168.230.174
403后端配置有错误
502后端
```

#### 观察nginx服务器的日志

```
[root@node1 logs]# pwd
/usr/local/nginx/logs
[root@node1 logs]# 
[root@node1 logs]# tail -f proxy_oa.qf2204.com.access.log
192.168.230.1 - - [03/May/2024:11:04:50 -0400] "GET / HTTP/1.1" 403 187 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 Edg/124.0.0.0" "-"
192.168.230.1 - - [03/May/2024:11:19:58 -0400] "GET / HTTP/1.1" 403 187 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 Edg/124.0.0.0" "-"
192.168.230.1 - - [03/May/2024:11:19:58 -0400] "GET / HTTP/1.1" 403 187 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 Edg/124.0.0.0" "-"

```









