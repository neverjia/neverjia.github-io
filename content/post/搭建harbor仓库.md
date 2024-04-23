---
title: "搭建harbor仓库"
slug: "搭建harbor仓库"
date: "2024-04-23"
created: "2024-04-23"
tags: ["harbor"]
---

### 简介
> Harbor是企业级的Docker Registry管理系统，由VMware公司开发并开源。作为一款专门为企业打造的容器镜像仓库解决方案


### 安装步骤
#### 解决依赖
##### 软件依赖
- Docker：确保目标主机已安装Docker，并且版本至少为18.09及以上版本。
- Docker Compose：同样需要安装Docker Compose，建议使用版本1.25或更高版本。
```
#使用命令检查是否安装相关依赖软件
[root@node ~]# docker-compose --version
[root@node ~]# docker --version

```
##### 网络和域名/IP
* 一个固定的域名或IP地址，用于从内部或外部访问Harbor registry服务。
* 若计划使用HTTPS加密连接，需提前准备SSL/TLS证书，确保Harbor的通信安全。

##### 防火墙和端口
- 开启必要的网络端口，如80用于HTTP服务（若未启用HTTPS则仅需80端口），443用于HTTPS服务，以及Harbor内部组件之间的通信端口。


安装harbor
从github下载harbor离线安装包：
> https://github.com/goharbor/harbor/releases/download/

```
wget https://github.com/goharbor/harbor/releases/download/v2.10.2/harbor-online-installer-v2.10.2.tgz

tar zxf harbor-online-installer-v2.10.2.tgz
```

#### 修改配置文件
```
#进入安装目录：
cd harbor

#在harbor.yml 中修改必要的配置项，如：
#        修改 hostname 为你的Harbor服务对外可见的域名或IP地址。
#        根据需要配置HTTP/HTTPS端口、数据库、数据存储位置和其他安全选项等。
#        修改默认密码

cp harbor.yml.tmpl harbor.yml
vi harbor.yml

#hostname: reg.mydomain.com #此处为你的IP
`hostname: 110.40.134.*`

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80

#https协议通信，如果不考虑对外访问，可注释
# https related config
#https:
#  # https port for harbor, default is 443
#  port: 443
#  # The path of cert and key files for nginx
#  certificate: /your/certificate/path
#  private_key: /your/private/key/path

#修改harbor的默认登录密码
#harbor_admin_password: Harbor12345
`harbor_admin_password: 修改为你自己的密码`

#数据存放的默认路径，可修改
data_volume: /data

```


#### 部署：
```
[root@node harbor]# pwd
/root/harbor
[root@node harbor]# ./prepare
[root@node harbor]# ./install.sh 

#install.sh安装分为4个步骤
loading Harbor images
preparing environment
checking existing instance of Harbor
starting Harbor

```
#### 安装完成提示：
```
[Step 4]: starting Harbor ...
WARN[0000] /root/harbor/docker-compose.yml: `version` is obsolete 
[+] Running 9/10
 ⠴ Network harbor_harbor        Created                                                2.5s 
 ✔ Container harbor-log         Started                                                0.7s 
 ✔ Container registry           Started                                                1.3s 
 ✔ Container registryctl        Started                                                1.3s 
 ✔ Container harbor-db          Started                                                1.0s 
 ✔ Container redis              Started                                                1.3s 
 ✔ Container harbor-portal      Started                                                1.0s 
 ✔ Container harbor-core        Started                                                1.6s 
 ✔ Container harbor-jobservice  Started                                                1.9s 
 ✔ Container nginx              Started                                                2.1s 
✔ ----Harbor has been installed and started successfully.----


```

#### 通过浏览器登录验证：
> http://你的内网IP:80
> 用户名：admin
> 密码
##### 查看harbor账密
> Harbor仓库的默认管理员账号通常是 admin，默认密码在不同版本和部署场景中可能有所不同，但常见的默认密码是 Harbor12345。例如可通过配置文件harbor.yml里面的 harbor_admin_password 参数设定，允许自定义默认密码。


#### 启动harbor仓库，都会运行以下这些容器
```
[root@node harbor]# docker ps -a
CONTAINER ID   IMAGE                                 COMMAND                  CREATED       STATUS                 PORTS                                   NAMES
dc7061052a76   goharbor/harbor-jobservice:v2.10.2    "/harbor/entrypoint.…"   6 hours ago   Up 6 hours (healthy)                                           harbor-jobservice
7a3605f02b90   goharbor/nginx-photon:v2.10.2         "nginx -g 'daemon of…"   6 hours ago   Up 6 hours (healthy)   0.0.0.0:80->8080/tcp, :::80->8080/tcp   nginx
609e26c91601   goharbor/harbor-core:v2.10.2          "/harbor/entrypoint.…"   6 hours ago   Up 6 hours (healthy)                                           harbor-core
43e51201aaf0   goharbor/registry-photon:v2.10.2      "/home/harbor/entryp…"   6 hours ago   Up 6 hours (healthy)                                           registry
1f0717c920b2   goharbor/harbor-portal:v2.10.2        "nginx -g 'daemon of…"   6 hours ago   Up 6 hours (healthy)                                           harbor-portal
f300cb5c1b22   goharbor/harbor-registryctl:v2.10.2   "/home/harbor/start.…"   6 hours ago   Up 6 hours (healthy)                                           registryctl
dcd8a9c3e871   goharbor/harbor-db:v2.10.2            "/docker-entrypoint.…"   6 hours ago   Up 6 hours (healthy)                                           harbor-db
15c7ddcc70cb   goharbor/redis-photon:v2.10.2         "redis-server /etc/r…"   6 hours ago   Up 6 hours (healthy)                                           redis
585b2a1b82c5   goharbor/harbor-log:v2.10.2           "/bin/sh -c /usr/loc…"   6 hours ago   Up 6 hours (healthy)   127.0.0.1:1514->10514/tcp               harbor-log



```


