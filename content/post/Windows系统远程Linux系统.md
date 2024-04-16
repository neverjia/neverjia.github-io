---
title: "Linux系统怎么被Windows远程"
slug: "k"
date: "2024-04-13"
created: "2024-04-13"
tags: ["OP"]
---

#检查是否安装epel库
rpm -qa  |  epel  
#安装软件包
yum -y install  epel-release
yum -y install  xrdp
yum -y install  tigervnc-server

#设置root免密
[root@node ~]# vncpasswd root
Password:
Verify:
Would you like to enter a view-only password (y/n)? y
Password:
Verify:

#关闭SELinux
[root@node ~]# /usr/sbin/sestatus -v
SELinux status:                 disabled
#临时关闭
setenforce 0
#关闭防火墙或者防火墙放行3389端口
[root@node ~]# systemctl status firewalld
○ firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; dis>
     Active: inactive (dead)
       Docs: man:firewalld(1)
        CPU: 0



#启动服务
[root@node ~]# systemctl enable xrdp
Created symlink /etc/systemd/system/multi-user.target.wants/xrdp.service → /usr/lib/systemd/system/xrdp.service.

#放行3389端口
firewall-cmd --permanent --zone=public --add-port=33898/tcp
#重新加载防火墙，使得配置生效
firewall-cmd  --reload

#启动xrdp服务，并且设置为开机自启动
[root@node ~]# systemctl status xrdp
● xrdp.service - xrdp daemon
     Loaded: loaded (/usr/lib/systemd/system/xrdp.service; enabled;>
     Active: active (running) since Wed 2024-01-03 00:01:16 CST; 13>
       Docs: man:xrdp(8)
             man:xrdp.ini(5)
   Main PID: 1442992 (xrdp)
      Tasks: 1 (limit: 10747)
     Memory: 2.0M
        CPU: 7ms
     CGroup: /system.slice/xrdp.service
             └─1442992 /usr/sbin/xrdp --nodaemon

Jan 03 00:01:16 node systemd[1]: Started xrdp daemon.
Jan 03 00:01:16 node xrdp[1442992]: [INFO ] starting xrdp with pid >
Jan 03 00:01:16 node xrdp[1442992]: [INFO ] address [0.0.0.0] port >
Jan 03 00:01:16 node xrdp[1442992]: [INFO ] listening to port 3389 >
Jan 03 00:01:16 node xrdp[1442992]: [INFO ] xrdp_listen_pp done



Win系统下“Win+R”键，	在弹出的“运行”框中输入“mstsc"命令，按”确认“，打开Windows远程