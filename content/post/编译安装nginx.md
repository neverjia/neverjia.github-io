配置yum源

`[centos镜像_centos下载地址_centos安装教程-阿里巴巴开源镜像站 (aliyun.com)](https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11BHBe7Y)`

```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo


或者
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo



 wget -O /etc/yum.repos.d/epel.repo  http://mirrors.aliyun.com/repo/epel-7.repo

```

编译nginx 开启ssl

`[nginx重新编译ssl模块详细教程_51CTO博客_nginx ssl模块](https://blog.51cto.com/u_13363488/2350495)`

yum -y install openssl openssl-devel pcre pcre-devel zlib zlib-devel

执行编译参数
#
 ./configure  --prefix=/usr/local/nginx --user=www --group=www --prefix=/usr/local/nginx  --with-http_ssl_module --with-http_stub_status_module

添加安装Openssl功能

```
 ./configure  --prefix=/usr/local/nginx   --with-http_ssl_module 
```

开始编译安装

```
make && make install
```

make成功以后会生成安装目录/usr/local/nginx

卸载只需要删除这个目录即可











缺失软件包

```
Error: Nothing to do
[root@liang ~]# yum-config-manager --enable rhel-7-server-optional-rpms
bash: yum-config-manager: command not found


Error: Nothing to do
[root@liang ~]# yum-config-manager --enable rhel-7-server-optional-rpms
bash: yum-config-manager: command not found

```

