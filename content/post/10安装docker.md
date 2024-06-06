---
title: "安装docker"
slug: "安装docker"
date: "2024-06-05"
created: "2024-06-05"
tags: ["volume"]
---



# 1.CentOS安装docker



## 1.卸载旧版本

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```





## 2.安装依赖

```
 yum install -y yum-utils
```

## 3.添加 `yum` 软件源

```
yum-config-manager     --add-repo     https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo
```

## 4.安装docker

```
 yum install docker-ce docker-ce-cli containerd.io
```

## 5.启动服务

```
systemctl enable docker
systemctl start docker
systemctl status docker
```

## 6.测试docker是否安装成功

```
#查看版本信息
[root@ntp-01 ~]# docker --version
Docker version 26.1.4, build 5650f9b

[root@ntp-01 ~]# docker run --rm hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete 
Digest: sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/



```





# 2.安装docker-compose

```
curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

[root@ntp-01 ~]# docker --version
Docker version 26.1.4, build 5650f9b
[root@ntp-01 ~]# docker-compose --version
docker-compose version 1.27.4, build 40524192
[root@ntp-01 ~]# ls -l /usr/local/bin/
total 11936
-rwxr-xr-x. 1 root root 12218968 Jun  7 09:07 docker-compose
[root@ntp-01 ~]# 

```



# 3.镜像加速

```
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://gkseqe2l.mirror.aliyuncs.com"]
}
EOF

systemctl daemon-reload
systemctl restart docker
```

