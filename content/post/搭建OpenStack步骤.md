---
title: "搭建 OpenStack"
slug: "搭建 OpenStack"
date: "2024-04-13"
created: "2024-04-13"
tags: ["OP"]
---


# 环境搭建（在线源安装）

Openstack环境搭建有很多种方法：

1.手工搭建，一个组件一个组件去安装。

2.通过工具 packstack 生成一个应答文件，编写应答文件，packstack调用文件去安装openstack环境。

3.HCS huaweicloud stack deploy 工具，私有云部署工具，类似tripple O的方式 openstack on openstack，首先通过安装有个精简版的openstack（heat编排服务），通过该服务去一键式安装整个云环境。



### 关闭防火墙selinux和networkmanager

控制节点和计算节点都要做

```
[root@controller ~]# systemctl stop firewalld

[root@controller ~]# systemctl disable firewalld

[root@controller ~]# setenforce 0

[root@controller ~]# sed -i  's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

[root@controller ~]# systemctl stop NetworkManager

[root@controller ~]# systemctl disable NetworkManager

[root@compute ~]# systemctl stop firewalld

[root@compute ~]# systemctl disable firewalld

[root@compute ~]# setenforce 0

[root@compute ~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

[root@compute ~]# systemctl stop NetworkManager

[root@compute ~]# systemctl disable NetworkManager
```

 

### **配置ip及主机名安装基础包**

控制节点和计算节点都要配置

```
[root@compute network-scripts]# cat ifcfg-ens160
TYPE=Ethernet
BOOTPROTO=none
NAME=ens160
DEVICE=ens160
ONBOOT=yes
IPADDR=192.168.230.139
NETMASK=255.255.255.0
GATEWAY=192.168.230.2
DNS1=192.168.230.2
[root@compute network-scripts]#
```

配置域名解析

- 在本地将域名指向自己想要解析的IP
- 当我们访问域名的时候  默认第一步就会先看本地的hosts文件  没有的话才会通过dns服务器获取解析（当然这个顺序是可以通过下面讲的host.conf改变的）

[root@controller ~]# vi /etc/hosts

添加下面两行

```
192.168.230.138 controller

192.168.230.139 compute
```

 发送给计算节点

```
scp  /etc/hosts  root@192.168.230.139:/etc/hosts
```



### **配置ntp时钟**

控制节点和计算节点都要配置

```
 yum install -y vim net-tools bash-completion chrony.x86_64 centos-release-openstack-victoria.noarch

[root@controller ~]# vim  /etc/chrony.conf

\# pool 2.centos.pool.ntp.org iburst

pool ntp.aliyun.com iburst

 

[root@compute ~]# vim /etc/chrony.conf 

\# pool 2.centos.pool.ntp.org iburst

pool ntp.aliyun.com iburst



启动服务

[root@controller ~]# systemctl start chronyd.service

[root@controller ~]# systemctl enable chronyd.service

 

[root@compute ~]# systemctl start chronyd.service

[root@compute ~]# systemctl enable chronyd.service

 
  ```

### **配置yum源**

 ```

控制节点和计算节点都要配置 

[root@controller ~]# mkdir /etc/yum.repos.d/bak

[root@controller ~]# mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak/

[root@controller ~]# ls /etc/yum.repos.d/

bak

[root@controller yum.repos.d]# cat cloudcs.repo 


[highavailability]

name=CentOS Stream 8 - HighAvailability

baseurl=https://mirrors.aliyun.com/centos/8-stream/HighAvailability/x86_64/os/

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial

gpgcheck=1

repo_gpgcheck=0

metadata_expire=6h

countme=1

enabled=1


[nfv]

name=CentOS Stream 8 - NFV

baseurl=https://mirrors.aliyun.com/centos/8-stream/NFV/x86_64/os/

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial

gpgcheck=1

repo_gpgcheck=0

metadata_expire=6h

countme=1

enabled=1

 

[rt]

name=CentOS Stream 8 - RT

baseurl=https://mirrors.aliyun.com/centos/8-stream/RT/x86_64/os/

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial

gpgcheck=1

repo_gpgcheck=0

metadata_expire=6h

countme=1

enabled=1

 

[resilientstorage]

name=CentOS Stream 8 - ResilientStorage

baseurl=https://mirrors.aliyun.com/centos/8-stream/ResilientStorage/x86_64/os/

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial

gpgcheck=1

repo_gpgcheck=0

metadata_expire=6h

countme=1

enabled=1

 

[extras-common]

name=CentOS Stream 8 - Extras packages

baseurl=https://mirrors.aliyun.com/centos/8-stream/extras/x86_64/extras-common/

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Extras-SHA512

gpgcheck=1

repo_gpgcheck=0

metadata_expire=6h

countme=1

enabled=1

 

[extras]

name=CentOS Stream  - Extras

mirrorlist=http://mirrorlist.centos.org/?release=&arch=&repo=extras&infra=

\#baseurl=http://mirror.centos.org///extras//os/

baseurl=https://mirrors.aliyun.com/centos/8-stream/extras/x86_64/os/

gpgcheck=1

enabled=1

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial

 

[centos-ceph-pacific]

name=CentOS - Ceph Pacific

baseurl=https://mirrors.aliyun.com/centos/8-stream/storage/x86_64/ceph-pacific/

gpgcheck=0

enabled=1

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Storage

 

[centos-rabbitmq-38]

name=CentOS-8 - RabbitMQ 38

baseurl=https://mirrors.aliyun.com/centos/8-stream/messaging/x86_64/rabbitmq-38/

gpgcheck=1

enabled=1

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Messaging

 

[centos-nfv-openvswitch]

name=CentOS Stream 8 - NFV OpenvSwitch

baseurl=https://mirrors.aliyun.com/centos/8-stream/nfv/x86_64/openvswitch-2/

gpgcheck=1

enabled=1

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-NFV

module_hotfixes=1

 

[baseos]

name=CentOS Stream 8 - BaseOS

baseurl=https://mirrors.aliyun.com/centos/8-stream/BaseOS/x86_64/os/

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial

gpgcheck=1

repo_gpgcheck=0

metadata_expire=6h

countme=1

enabled=1

 

[appstream]

name=CentOS Stream 8 - AppStream

baseurl=https://mirrors.aliyun.com/centos/8-stream/AppStream/x86_64/os/

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial

gpgcheck=1

repo_gpgcheck=0

metadata_expire=6h

countme=1

enabled=1

 

[centos-openstack-victoria]

name=CentOS 8 - OpenStack victoria

baseurl=https://mirrors.aliyun.com/centos/8-stream/cloud/x86_64/openstack-victoria/

\#baseurl=https://repo.huaweicloud.com/centos/8-stream/cloud/x86_64/openstack-yoga/

gpgcheck=1

enabled=1

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Cloud

module_hotfixes=1

 

[powertools]

name=CentOS Stream 8 - PowerTools

\#mirrorlist=http://mirrorlist.centos.org/?release=&arch=&repo=PowerTools&infra=

baseurl=https://mirrors.aliyun.com/centos/8-stream/PowerTools/x86_64/os/

gpgcheck=1

enabled=1

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
```
 ```
 

将控制节点的repo文件拷贝过来到计算节点

[root@compute yum.repos.d]# mkdir bak

[root@compute yum.repos.d]# mv *.repo bak/

[root@compute yum.repos.d]# ls

bak

[root@compute yum.repos.d]# scp controller:/etc/yum.repos.d/cloudcs.repo  .

[root@compute yum.repos.d]# ls

bak  cloudcs.repo

 

[root@compute yum.repos.d]# yum clean all

27 files removed

[root@compute yum.repos.d]# yum repolist all

 

### **安装并配置packstack**

仅在控制节点上做。

 

安装packstack工具

[root@controller ~]# yum install -y openstack-packstack 

生成应答文件

[root@controller ~]# packstack --help |grep ans

 --gen-answer-file=GEN_ANSWER_FILE

​            Generate a template of an answer file.

 --validate-answer-file=VALIDATE_ANSWER_FILE

​            Check if answerfile contains unexpected options.

 --answer-file=ANSWER_FILE

​            answerfile will also be generated and should be used

 -o, --options     Print details on options available in answer file(rst

​            Packstack a second time with the same answer file and

​            attribute where "y" means an account is disabled.

  --manila-netapp-transport-type=MANILA_NETAPP_TRANSPORT_TYPE

​            The transport protocol used when communicating with

[root@controller ~]# 

[root@controller ~]# 

[root@controller ~]# packstack --gen-answer-file=memeda.txt

[root@controller ~]# ls

anaconda-ks.cfg  memeda.txt

 

编辑应答文件

[root@controller ~]# vim memeda.txt 

 \#修改以下参数内容

CONFIG_COMPUTE_HOSTS=192.168.230.138,192.168.230.139

CONFIG_KEYSTONE_ADMIN_PW=redhat

CONFIG_PROVISION_DEMO=n

CONFIG_HEAT_INSTALL=y

CONFIG_NEUTRON_OVN_BRIDGE_IFACES=br-ex:ens160

 

***\*需要注意的是 br-ex 后面对应的物理网卡名称\****

 

### **安装openstack**

```
packstack --answer-file=memeda.txt  #（等大约十几分钟）

```
admin

密码

```
### 显示如下代表搭建成功
```
Applying Puppet manifests                            [ DONE ]
Finalizing                                           [ DONE ]

 **** Installation completed successfully ******

Additional information:
 * Parameter CONFIG_NEUTRON_L2_AGENT: You have chosen OVN Neutron backend. Note that this backend does not support the VPNaaS plugin. Geneve will be used as the encapsulation method for tenant networks
 * Time synchronization installation was skipped. Please note that unsynchronized time on server instances might be problem for some OpenStack components.
 * File /root/keystonerc_admin has been created on OpenStack client host 192.168.230.138. To use the command line tools you need to source the file.
 * To access the OpenStack Dashboard browse to http://192.168.230.138/dashboard .
Please, find your login credentials stored in the keystonerc_admin in your home directory.
 * The installation log file is available at: /var/tmp/packstack/20231007-150224-_3m58aw_/openstack-setup.log
 * The generated manifests are available at: /var/tmp/packstack/20231007-150224-_3m58aw_/manifests
[root@controller ~]#
```