---
title: "编译安装OpenSSH"
slug: "编译安装OpenSSH"
date: "2024-04-16"
created: "2024-04-13"
tags: ["OP"]
---


## 检查主机版本信息

```
[root@node1 ~]# cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)
[root@node1 ~]# openssl version
OpenSSL 1.0.2k-fips  26 Jan 2017
[root@node1 ~]# ssh -V
OpenSSH_7.4p1, OpenSSL 1.0.2k-fips  26 Jan 2017

```

相关网站链接
https://www.openssl.org/source
https://wwww.openssh.com/

http://rpmfind.net




## 该命令的含义是以自动确认的方式安装名为 "Development Tools" 的软件包组。这通常用于设置开发环境，其中包括编译器、调试器和其他开发工具，以便用户能够编译和构建软件。

[root@node1 ~]# yum groupinstall "Development Tools" -y

## 名称是 perl-core，表示安装 Perl 语言的核心模块和依赖项。

[root@node1 ~]# yum -y install perl-core 



[root@node1 ~]# yum install gcc gcc-c++ glibc make autoconf openssl-devel pcre-devel pam-devel perl-core

## 卸载旧的版本

[root@node1 ~]# yum remove -y openssl


wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.7p1.tar.gz
wget https://www.openssl.org/source/old/1.1.1/openssl-1.1.1w.tar.gz

```
[root@node1 ~]# wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.7p1.tar.gz
--2024-04-02 10:40:11--  https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.7p1.tar.gz
Resolving cdn.openbsd.org (cdn.openbsd.org)... 146.75.115.52, 2a04:4e42:8c::820
Connecting to cdn.openbsd.org (cdn.openbsd.org)|146.75.115.52|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1848766 (1.8M) [application/octet-stream]
Saving to: ‘openssh-9.7p1.tar.gz’

100%[===================================================================================================================>] 1,848,766   58.1KB/s   in 37s    

2024-04-02 10:40:49 (48.9 KB/s) - ‘openssh-9.7p1.tar.gz’ saved [1848766/1848766]

[root@node1 ~]# wget https://ww.openssl.org/source/old/1.1.1/openssl-1.1.1w.tar.gz
--2024-04-02 10:40:54--  https://ww.openssl.org/source/old/1.1.1/openssl-1.1.1w.tar.gz
Resolving ww.openssl.org (ww.openssl.org)... failed: Name or service not known.
wget: unable to resolve host address ‘ww.openssl.org’
[root@node1 ~]# wget https://www.openssl.org/source/old/1.1.1/openssl-1.1.1w.tar.gz
--2024-04-02 10:41:02--  https://www.openssl.org/source/old/1.1.1/openssl-1.1.1w.tar.gz
Resolving www.openssl.org (www.openssl.org)... 34.36.58.177, 2600:1901:0:1812::
Connecting to www.openssl.org (www.openssl.org)|34.36.58.177|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9893384 (9.4M) [application/x-tar]
Saving to: ‘openssl-1.1.1w.tar.gz’

100%[===================================================================================================================>] 9,893,384   26.1KB/s   in 2m 35s 

2024-04-02 10:43:40 (62.1 KB/s) - ‘openssl-1.1.1w.tar.gz’ saved [9893384/9893384]

```

[root@node1 ~]# ll
total 11476
-rw-------. 1 root root    1244 Nov  1 02:15 anaconda-ks.cfg
-rw-r--r--  1 root root 1848766 Mar 11 06:19 openssh-9.7p1.tar.gz
-rw-r--r--  1 root root 9893384 Jan 30 09:48 openssl-1.1.1w.tar.gz

解压文件
root@node1 ~]# tar -zxvf openssh-9.7p1.tar.gz 
[root@node1 ~]# tar -zxvf openssl-1.1.1w.tar.gz 


[root@node1 ~]# ls
anaconda-ks.cfg  openssh-9.7p1  openssh-9.7p1.tar.gz  openssl-1.1.1w  openssl-1.1.1w.tar.gz
[root@node1 ~]# cd openssl-1.1.1w/
[root@node1 openssl-1.1.1w]# ls
ACKNOWLEDGEMENTS  CHANGES         Configure     doc       FAQ      LICENSE        NOTES.DJGPP  NOTES.WIN      README.FIPS  util
apps              config          CONTRIBUTING  engines   fuzz     ms             NOTES.PERL   os-dep         ssl          VMS
AUTHORS           config.com      crypto        e_os.h    include  NEWS           NOTES.UNIX   README         test         wycheproof
build.info        Configurations  demos         external  INSTALL  NOTES.ANDROID  NOTES.VMS    README.ENGINE  tools
[root@node1 openssl-1.1.1w]# ./config --prefix=/usr/local/ssl shared
Operating system: x86_64-whatever-linux2
Configuring OpenSSL version 1.1.1w (0x1010117fL) for linux-x86_64
Using os-specific seed configuration
Creating configdata.pm
Creating Makefile

**********************************************************************

***                                                                ***

***   OpenSSL has been successfully configured                     ***

***                                                                ***

***   If you encounter a problem while building, please open an    ***
***   issue on GitHub <https://github.com/openssl/openssl/issues>  ***
***   and include the output from the following command:           ***

***                                                                ***

***       perl configdata.pm --dump                                ***

***                                                                ***

***   (If you are new to OpenSSL, you might want to consult the    ***
***   'Troubleshooting' section in the INSTALL file first)         ***

***                                                                ***

**********************************************************************

[root@node1 openssl-1.1.1w]# 

```
make #编译-->预计时间3-5分钟
make test #测试-->其中会出现几项skiped可以忽略，只要最后看到All tests successful.、Result:PASS即可
make install #安装-->预计时间1-2分钟
echo /usr/local/ssl/lib/>>/etc/ld.so.conf.d/common.conf #配置so文件的位置
ldconfig #启用so文件位置的配置
mv /usr/bin/openssl /usr/bin/openssl.bak #报错No such file or directory可以忽略
ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/ssl/include/openssl /usr/include/openssl
[root@node1 openssl-1.1.1w]# openssl version
OpenSSL 1.1.1w  11 Sep 2023
[root@node1 openssl-1.1.1w]# 
```


## 安装openssh

cp -r /etc/ssh /root/
cd openssh-9.7p1/
yum install zlib* pam-devel -y
./configure --prefix=/usr --sysconfdir=/etc/ssh --with-ssl-dir=/usr/local/ssl --with-pam #检查一>预计时间1-2分钟



make #编译-->预计时间1-2分钟
make install #若最后结果提示文件权限过大()，执行命令:chmod 0600 <file_name>即可
ssh -V
[root@node1 openssh-9.7p1]# ssh -V
\OpenSSH_9.7p1, OpenSSL 1.1.1w  11 Sep 2023
[root@node1 openssh-9.7p1]# \

cp /usr/lib/systemd/system/sshd.service /usr/lib/systemd/system/sshd.service.bak

[root@node1 openssh-9.7p1]# cat /usr/lib/systemd/system/sshd.service
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8)man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service

[Service]
Type=simple
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s
[Install]
WantedBy=multi-user.target
[root@node1 openssh-9.7p1]# 


#systemctl restart sshd


## 错误记录

[root@node1 openssh-9.7p1]# systemctl status sshd.service
● sshd.service - 0penSSH server daemon
   Loaded: error (Reason: Bad message)
   Active: failed (Result: resources) since Tue 2024-04-02 11:24:00 EDT; 16s ago
     Docs: man:sshd(8)man:sshd_config(5)
 Main PID: 73916 (code=exited, status=1/FAILURE)

Apr 02 11:23:48 node1 sshd[73916]: sshd: no hostkeys available -- exiting.
Apr 02 11:23:48 node1 systemd[1]: sshd.service: main process exited, code=exited, status=1/FAILURE
Apr 02 11:23:48 node1 systemd[1]: Failed to start OpenSSH server daemon.
Apr 02 11:23:48 node1 systemd[1]: Unit sshd.service entered failed state.
Apr 02 11:23:48 node1 systemd[1]: sshd.service failed.
Apr 02 11:23:59 node1 systemd[1]: [/usr/lib/systemd/system/sshd.service:7] Missing '='.
Apr 02 11:24:00 node1 systemd[1]: sshd.service holdoff time over, scheduling restart.
Apr 02 11:24:00 node1 systemd[1]: sshd.service failed to schedule restart job: Unit is not loaded properly: Bad message.
Apr 02 11:24:00 node1 systemd[1]: Unit sshd.service entered failed state.
Apr 02 11:24:00 node1 systemd[1]: sshd.service failed.



①配置文件配置错了
②Apr 02 11:38:20 node1 systemd[1]: sshd.service: main process exited, code=exited, status=1/FAILURE
Apr 02 11:38:20 node1 systemd[1]: Unit sshd.service entered failed state.
Apr 02 11:38:20 node1 systemd[1]: sshd.service failed.


命令
journalctl -u sshd.service
Apr 02 11:23:48 node1 systemd[1]: Failed to start OpenSSH server daemon.


Apr 02 11:39:02 node1 sshd[74012]: sshd: no hostkeys available -- exiting.
Apr 02 11:39:02 node1 systemd[1]: Unit sshd.service entered failed state.
Apr 02 11:39:02 node1 systemd[1]: sshd.service failed.




## 错误

```
Apr 02 11:41:04 node1 sshd[74050]: Permissions 0640 for '/etc/ssh/ssh_host_ed25519_key' are too open.
Apr 02 11:41:04 node1 sshd[74050]: It is required that your private key files are NOT accessible by others.
Apr 02 11:41:04 node1 sshd[74050]: This private key will be ignored.
Apr 02 11:41:04 node1 sshd[74050]: Unable to load host key "/etc/ssh/ssh_host_ed25519_key": bad permissions
Apr 02 11:41:04 node1 sshd[74050]: Unable to load host key: /etc/ssh/ssh_host_ed25519_key
Apr 02 11:41:04 node1 sshd[74050]: sshd: no hostkeys available -- exiting.
Apr 02 11:41:04 node1 systemd[1]: sshd.service: main process exited, code=exited, status=1/FAILURE
Apr 02 11:41:04 node1 systemd[1]: Unit sshd.service entered failed state.
Apr 02 11:41:04 node1 systemd[1]: sshd.service failed.
[root@node1 openssh-9.7p1]# ssh -V

## 解决办法
[root@node1 openssh-9.7p1]# sudo chmod 600 /etc/ssh/ssh_host_ed25519_key
[root@node1 openssh-9.7p1]# sudo systemctl daemon-reload
[root@node1 openssh-9.7p1]# sudo systemctl restart sshd.service
[root@node1 openssh-9.7p1]# systemctl status sshd
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2024-04-02 11:43:21 EDT; 4s ago
     Docs: man:sshd(8)man:sshd_config(5)
 Main PID: 74087 (sshd)
   CGroup: /system.slice/sshd.service
           └─74087 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups

Apr 02 11:43:21 node1 sshd[74087]: Unable to load host key: /etc/ssh/ssh_host_rsa_key
Apr 02 11:43:21 node1 sshd[74087]: @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Apr 02 11:43:21 node1 sshd[74087]: @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
Apr 02 11:43:21 node1 sshd[74087]: @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Apr 02 11:43:21 node1 sshd[74087]: Permissions 0640 for '/etc/ssh/ssh_host_ecdsa_key' are too open.
Apr 02 11:43:21 node1 sshd[74087]: It is required that your private key files are NOT accessible by others.
Apr 02 11:43:21 node1 sshd[74087]: This private key will be ignored.
Apr 02 11:43:21 node1 sshd[74087]: Unable to load host key "/etc/ssh/ssh_host_ecdsa_key": bad permissions
Apr 02 11:43:21 node1 sshd[74087]: Unable to load host key: /etc/ssh/ssh_host_ecdsa_key
Apr 02 11:43:21 node1 sshd[74087]: Server listening on :: port 22.
[root@node1 openssh-9.7p1]# 




```





[root@node1 openssh-9.7p1]# ssh -V
OpenSSH_9.7p1, OpenSSL 1.1.1w  11 Sep 2023






备注

```
yum: 是一种在基于 RPM 包管理的 Linux 发行版中使用的包管理器，它允许用户从软件仓库中安装、更新和删除软件包。
install: 是 yum 命令的一个子命令，用于安装软件包。
gcc: 是 GNU Compiler Collection 的缩写，是一套开源的编译器工具集，用于编译和构建 C 语言程序。
gcc-c++: 是 GNU Compiler Collection 的 C++ 编译器。
glibc: 是 GNU C Library 的缩写，是一套 C 语言库，提供了对操作系统核心功能的访问接口。
make: 是一个工具，用于自动化源代码编译和构建过程。
autoconf: 是一个工具，用于自动化生成软件包的配置脚本。
openssl-devel: 是 OpenSSL 的开发库，用于支持 SSL 和加密功能。
pcre-devel: 是 Perl Compatible Regular Expressions (PCRE) 的开发库，用于支持正则表达式功能。
pam-devel: 是 Pluggable Authentication Modules (PAM) 的开发库，用于支持认证和访问控制功能。
perl-core: 是 Perl 语言的核心模块和依赖项。
```





系统基础优化

1)系统用户管理优化

2)系统命令提示符优化
3)系统安全服务优化
4)系统yum源优化
  安装系统基础软件包
5系统字符集优化

6系统时间同步优化

02课程知识回顾
1)系统安装软件方式4种(yum  rpm 编译   二进制)

2)系统常见日志文件  2个       messages系统服务运行状况  secure记录用户登录信息

3)系统硬件信息查看

CPU   lsblk   /proc/cpuinfo

内存   free -h  /proc/meminfo

磁盘  df  -h     /proc/mounts

负载  w    /proc/loadavg





1)系统用户管理优化2)系统命令提示符优化
3)系统安全服务优化
4)系统yum源优化
安装系统基础软件包
56系统字符集优化系统时间同步优化
02课程知识回顾
1)系统安装软件方式4种(yumrpm 编译二进制)2)系统常见日志文件2个消息系统服务运行状况







有多少个CPU

```
[root@node5 ~]# grep  "physical id"  /proc/cpuinfo
physical id     : 0
physical id     : 0
[root@node5 ~]#
```





去重

[root@node5 ~]# cat test.txt
a
a
a
a
a
a
b
c
d
e
b
c
d
e
b
c
d
e
b
c
d
e
b
c
d
e
b
c
d
e
[root@node5 ~]# uniq -c  test.txt
      6 a
      1 b
      1 c
      1 d
      1 e
      1 b
      1 c
      1 d
      1 e
      1 b
      1 c
      1 d
      1 e
      1 b
      1 c
      1 d
      1 e
      1 b
      1 c
      1 d
      1 e
      1 b
      1 c
      1 d
      1 e
[root@node5 ~]#





系统查看系统版本和内核信息

uname -r                           #内核

cat  /etc/redhat-release  #版本

```
[root@node5 ~]# uname -r
3.10.0-1160.el7.x86_64
[root@node5 ~]# cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
[root@node5 ~]#



[root@node5 ~]# uname -a
Linux node5 3.10.0-1160.el7.x86_64 #1 SMP Mon Oct 19 16:18:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

```

 



服务

rsync备份服务

centos6是