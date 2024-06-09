---
title: "部署python3"
slug: "python3"
date: "2024-05-26"
created: "2024-05-26"
tags: ["python3"]
---
# Python3部署

### 1.准备环境

```
#关闭防火墙和selinux
[root@base-01 ~]# systemctl stop firewalld
[root@base-01 ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.

[root@base-01 ~]# vi /etc/sysconfig/selinux 
将文件中的SELINUX=enforcing改为disabled
[root@base-01 ~]# getenforce    #查看selinux是否开启
Enforcing
[root@base-01 ~]# setenforce 0  #临时关闭selinux



#检查服务端口使用情况
\# yum install net-tools

[root@master-61 ~]# netstat -tunlp 
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1044/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1292/master         
tcp6       0      0 :::22                   :::*                    LISTEN      1044/sshd           
tcp6       0      0 ::1:25                  :::*                    LISTEN      1292/master         
udp        0      0 0.0.0.0:68              0.0.0.0:*                           856/dhclient        
udp        0      0 127.0.0.1:323           0.0.0.0:*                           705/chronyd         
udp6       0      0 ::1:323                 :::*                                705/chronyd         
[root@master-61 ~]# 
```



### 1.yum源配置

```
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo  https://mirrors.aliyun.com/repo/epel-7.repo

yum clean all
yum makecache

```

### 2.部署跳板机的依赖软件


```
#1.安装基础环境软件
yum -y install bash-completion vim lrzsz wget expect net-tools nc nmap tree dos2unix htop iftop iotop unzip telnet sl psmisc nethogs glances bc ntpdate openldap-devel 

#git          ---用于下载jumpserver软件程序
python-pip    ---用于安装python软件
gcc           ---解析代码中c语言信息（解释器）
automake      ---实现软件自动编译过程
autoconf      ---实现软件自动配置过程
python-devel  ---在操作python命令可以实现补全

# 2.安装python程序，必须的基础依赖
yum -y install git python-pip gcc automake autoconf python-devel vim sshpass lrzsz readline-devel zlib zlib-devel openssl openssl-devel

# 3.python3开发的源代码，jumpserver，因此需要配置Python环境，编译安装，对版本有要求，
# 编译安装，Python可以处理ssl加密，需要安装openssl基础环境


```
### 3.设置系统编码环境

```
#设置主机的系统编码环境，支持中文
#错误写法
#localedef -c -f UTF-8 -i zh_CN.UTF-8
#正确写法
localedef -c -f UTF-8 -i zh_CN /usr/lib/locale/zh_CN.UTF-8

#设置操作系统所有的语言环境，改为中文utf8编码
export LC_ALL=zh_CN.UTF-8

#查看编码情况
#使用locale查看系统的所有编码情况，切换中英文支持时候需要设置

[root@master-61 ~]# echo $LC_ALL
zh_CN.UTF-8
[root@master-61 ~]# locale
LANG=en_US.UTF-8
LC_CTYPE="zh_CN.UTF-8"
LC_NUMERIC="zh_CN.UTF-8"
LC_TIME="zh_CN.UTF-8"
LC_COLLATE="zh_CN.UTF-8"
LC_MONETARY="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.UTF-8"
LC_PAPER="zh_CN.UTF-8"
LC_NAME="zh_CN.UTF-8"
LC_ADDRESS="zh_CN.UTF-8"
LC_TELEPHONE="zh_CN.UTF-8"
LC_MEASUREMENT="zh_CN.UTF-8"
LC_IDENTIFICATION="zh_CN.UTF-8"
LC_ALL=zh_CN.UTF-8
```

### 4.部署core服务

```
#1.下载jumpserver后端源码
mkdir /opt/jumpserver-v2.12.0
wget -O /opt/jumpserver-v2.12.0.tar.gz https://github.com/jumpserver/archive/refs/tags/v2.12.0.tar.gz

#2.解压缩源代码，安装后端源码，运行所需的环境依赖
#参数--strip-components 1意思是，直接解压其中的每一个文件，到/opt/jumpserver-v2.12.0
#原本的tar -xf直接解压会生成相应的文件夹，源代码都在这
-C 指定解压到的目录
tar -xf jumpserver-2.12.0.tar.gz -C /opt/jumpserver-v2.12.0 --strip-components 1

#运维开发和纯开发区别
#纯开发，纯后端，写的是业务代码，如电商后台，员工管理系统后台，医疗后台，管理的是纯业务的员工数据，商业数据
# 运维开发，写代码，写平台，数据库管理的是机器的信息，如资产管理平台，代码发布平台，容器管理平台你
#ansible自动化调度平台，通过运维技术开发技术，管理服务器等资产信息。


#3.检查解压的源码（这些是python的web框架，Django的源代码）
[root@master-61 jumpserver-v2.12.0]# pwd
/opt/jumpserver-v2.12.0
[root@master-61 jumpserver-v2.12.0]# ll
总用量 84
drwxrwxr-x. 20 root root  4096 7月  15 2021 apps
-rw-rw-r--.  1 root root  4283 7月  15 2021 config_example.yml
drwxrwxr-x.  3 root root    54 7月  15 2021 data
-rw-rw-r--.  1 root root  1680 7月  15 2021 Dockerfile
drwxrwxr-x.  2 root root    23 7月  15 2021 docs
-rwxrwxr-x.  1 root root   319 7月  15 2021 entrypoint.sh
-rwxrwxr-x.  1 root root 15675 7月  15 2021 jms
-rw-rw-r--.  1 root root 18045 7月  15 2021 LICENSE
drwxrwxr-x.  2 root root    22 7月  15 2021 logs
-rw-rw-r--.  1 root root  5648 7月  15 2021 README_EN.md
-rw-rw-r--.  1 root root  5818 7月  15 2021 README.md
drwxrwxr-x.  2 root root   198 7月  15 2021 requirements
-rw-rw-r--.  1 root root   212 7月  15 2021 run_server.py
drwxrwxr-x.  2 root root    22 7月  15 2021 tmp
drwxrwxr-x.  4 root root  4096 7月  15 2021 utils
-rw-rw-r--.  1 root root  1969 7月  15 2021 Vagrantfile
[root@master-61 jumpserver-v2.12.0]# 
```

#### 查看目录结构

```
#就是一些py代码，和sh脚本，这个源码是飞致云运维写的程序
[root@master-61 jumpserver-v2.12.0]# tree
.
├── apps
│   ├── acls
│   │   ├── admin.py
│   │   ├── api
│   │   │   ├── __init__.py
│   │   │   ├── login_acl.py
│   │   │   ├── login_asset_acl.py
│   │   │   └── login_asset_check.py
│   │   ├── apps.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   └── __init__.py
│   │   ├── models
│   │   │   ├── base.py
│   │   │   ├── __init__.py
│   │   │   ├── login_acl.py
│   │   │   └── login_asset_acl.py
│   │   ├── serializers
│   │   │   ├── __init__.py
│   │   │   ├── login_acl.py
│   │   │   ├── login_asset_acl.py
│   │   │   └── login_asset_check.py
│   │   ├── tests.py
│   │   ├── urls
│   │   │   ├── api_urls.py
│   │   │   └── __init__.py
│   │   └── utils.py
│   ├── applications
│   │   ├── admin.py
│   │   ├── api
│   │   │   ├── application.py
│   │   │   ├── application_user.py
│   │   │   ├── __init__.py
│   │   │   ├── mixin.py
│   │   │   └── remote_app.py
│   │   ├── apps.py
│   │   ├── const.py
│   │   ├── hands.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   ├── 0002_remove_remoteapp_system_user.py
│   │   │   ├── 0003_auto_20191210_1659.py
│   │   │   ├── 0004_auto_20191218_1705.py
│   │   │   ├── 0005_k8sapp.py
│   │   │   ├── 0006_application.py
│   │   │   ├── 0007_auto_20201119_1110.py
│   │   │   ├── 0008_auto_20210104_0435.py
│   │   │   ├── 0009_applicationuser.py
│   │   │   └── __init__.py
│   │   ├── models
│   │   │   ├── application.py
│   │   │   └── __init__.py
│   │   ├── permissions.py
│   │   ├── serializers
│   │   │   ├── application.py
│   │   │   ├── attrs
│   │   │   │   ├── application_category
│   │   │   │   │   ├── cloud.py
│   │   │   │   │   ├── db.py
│   │   │   │   │   ├── __init__.py
│   │   │   │   │   └── remote_app.py
│   │   │   │   ├── application_type
│   │   │   │   │   ├── chrome.py
│   │   │   │   │   ├── custom.py
│   │   │   │   │   ├── __init__.py
│   │   │   │   │   ├── k8s.py
│   │   │   │   │   ├── mariadb.py
│   │   │   │   │   ├── mysql.py
│   │   │   │   │   ├── mysql_workbench.py
│   │   │   │   │   ├── oracle.py
│   │   │   │   │   ├── pgsql.py
│   │   │   │   │   └── vmware_client.py
│   │   │   │   ├── attrs.py
│   │   │   │   └── __init__.py
│   │   │   ├── __init__.py
│   │   │   └── remote_app.py
│   │   ├── tests.py
│   │   └── urls
│   │       ├── api_urls.py
│   │       └── __init__.py
│   ├── assets
│   │   ├── api
│   │   │   ├── accounts.py
│   │   │   ├── admin_user.py
│   │   │   ├── asset.py
│   │   │   ├── cmd_filter.py
│   │   │   ├── domain.py
│   │   │   ├── favorite_asset.py
│   │   │   ├── gathered_user.py
│   │   │   ├── __init__.py
│   │   │   ├── label.py
│   │   │   ├── mixin.py
│   │   │   ├── node.py
│   │   │   ├── system_user.py
│   │   │   └── system_user_relation.py
│   │   ├── apps.py
│   │   ├── backends
│   │   │   └── db.py
│   │   ├── const.py
│   │   ├── exceptions.py
│   │   ├── filters.py
│   │   ├── hands.py
│   │   ├── __init__.py
│   │   ├── locks.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   ├── 0002_auto_20180105_1807.py
│   │   │   ├── 0002_auto_20180105_1807_squashed_0009_auto_20180307_1212.py
│   │   │   ├── 0003_auto_20180109_2331.py
│   │   │   ├── 0004_auto_20180125_1218.py
│   │   │   ├── 0005_auto_20180126_1637.py
│   │   │   ├── 0006_auto_20180130_1502.py
│   │   │   ├── 0007_auto_20180225_1815.py
│   │   │   ├── 0008_auto_20180306_1804.py
│   │   │   ├── 0009_auto_20180307_1212.py
│   │   │   ├── 0010_auto_20180307_1749.py
│   │   │   ├── 0010_auto_20180307_1749_squashed_0019_auto_20180816_1320.py
│   │   │   ├── 0011_auto_20180326_0957.py
│   │   │   ├── 0012_auto_20180404_1302.py
│   │   │   ├── 0013_auto_20180411_1135.py
│   │   │   ├── 0014_auto_20180427_1245.py
│   │   │   ├── 0015_auto_20180510_1235.py
│   │   │   ├── 0016_auto_20180511_1203.py
│   │   │   ├── 0017_auto_20180702_1415.py
│   │   │   ├── 0018_auto_20180807_1116.py
│   │   │   ├── 0019_auto_20180816_1320.py
│   │   │   ├── 0020_auto_20180816_1652.py
│   │   │   ├── 0021_auto_20180903_1132.py
│   │   │   ├── 0022_auto_20181012_1717.py
│   │   │   ├── 0023_auto_20181016_1650.py
│   │   │   ├── 0024_auto_20181219_1614.py
│   │   │   ├── 0025_auto_20190221_1902.py
│   │   │   ├── 0026_auto_20190325_2035.py
│   │   │   ├── 0027_auto_20190521_1703.py
│   │   │   ├── 0028_protocol.py
│   │   │   ├── 0029_auto_20190522_1114.py
│   │   │   ├── 0030_auto_20190619_1135.py
│   │   │   ├── 0031_auto_20190621_1332.py
│   │   │   ├── 0032_auto_20190624_2108.py
│   │   │   ├── 0033_auto_20190624_2108.py
│   │   │   ├── 0034_auto_20190705_1348.py
│   │   │   ├── 0035_auto_20190711_2018.py
│   │   │   ├── 0036_auto_20190716_1535.py
│   │   │   ├── 0037_auto_20190724_2002.py
│   │   │   ├── 0038_auto_20190911_1634.py
│   │   │   ├── 0039_authbook_is_active.py
│   │   │   ├── 0040_auto_20190917_2056.py
│   │   │   ├── 0041_gathereduser.py
│   │   │   ├── 0042_favoriteasset.py
│   │   │   ├── 0043_auto_20191114_1111.py
│   │   │   ├── 0044_platform.py
│   │   │   ├── 0045_auto_20191206_1607.py
│   │   │   ├── 0046_auto_20191218_1705.py
│   │   │   ├── 0047_assetuser.py
│   │   │   ├── 0048_auto_20191230_1512.py
│   │   │   ├── 0049_systemuser_sftp_root.py
│   │   │   ├── 0050_auto_20200711_1740.py
│   │   │   ├── 0051_auto_20200713_1143.py
│   │   │   ├── 0052_auto_20200715_1535.py
│   │   │   ├── 0053_auto_20200723_1232.py
│   │   │   ├── 0054_auto_20200807_1032.py
│   │   │   ├── 0055_auto_20200811_1845.py
│   │   │   ├── 0056_auto_20200904_1751.py
│   │   │   ├── 0057_fill_node_value_assets_amount_and_parent_key.py
│   │   │   ├── 0058_auto_20201023_1115.py
│   │   │   ├── 0059_auto_20201027_1905.py
│   │   │   ├── 0060_node_full_value.py
│   │   │   ├── 0061_auto_20201116_1757.py
│   │   │   ├── 0062_auto_20201117_1938.py
│   │   │   ├── 0063_migrate_default_node_key.py
│   │   │   ├── 0064_auto_20201203_1100.py
│   │   │   ├── 0065_auto_20210121_1549.py
│   │   │   ├── 0066_auto_20210208_1802.py
│   │   │   ├── 0067_auto_20210311_1113.py
│   │   │   ├── 0068_auto_20210312_1455.py
│   │   │   ├── 0069_change_node_key0_to_key1.py
│   │   │   ├── 0070_auto_20210426_1515.py
│   │   │   ├── 0071_systemuser_type.py
│   │   │   ├── 0072_historicalauthbook.py
│   │   │   ├── 0073_auto_20210606_1142.py
│   │   │   ├── 0074_remove_systemuser_assets.py
│   │   │   ├── 0075_auto_20210705_1759.py
│   │   │   ├── 0076_delete_assetuser.py
│   │   │   └── __init__.py
│   │   ├── models
│   │   │   ├── asset.py
│   │   │   ├── authbook.py
│   │   │   ├── base.py
│   │   │   ├── cluster.py
│   │   │   ├── cmd_filter.py
│   │   │   ├── domain.py
│   │   │   ├── favorite_asset.py
│   │   │   ├── gathered_user.py
│   │   │   ├── group.py
│   │   │   ├── __init__.py
│   │   │   ├── label.py
│   │   │   ├── node.py
│   │   │   ├── user.py
│   │   │   └── utils.py
│   │   ├── pagination.py
│   │   ├── serializers
│   │   │   ├── account.py
│   │   │   ├── admin_user.py
│   │   │   ├── asset.py
│   │   │   ├── base.py
│   │   │   ├── cmd_filter.py
│   │   │   ├── domain.py
│   │   │   ├── favorite_asset.py
│   │   │   ├── gathered_user.py
│   │   │   ├── __init__.py
│   │   │   ├── label.py
│   │   │   ├── node.py
│   │   │   └── system_user.py
│   │   ├── signals_handler
│   │   │   ├── asset.py
│   │   │   ├── authbook.py
│   │   │   ├── common.py
│   │   │   ├── __init__.py
│   │   │   ├── node_assets_amount.py
│   │   │   ├── node_assets_mapping.py
│   │   │   └── system_user.py
│   │   ├── tasks
│   │   │   ├── account_connectivity.py
│   │   │   ├── asset_connectivity.py
│   │   │   ├── common.py
│   │   │   ├── const.py
│   │   │   ├── gather_asset_hardware_info.py
│   │   │   ├── gather_asset_users.py
│   │   │   ├── __init__.py
│   │   │   ├── nodes_amount.py
│   │   │   ├── push_system_user.py
│   │   │   ├── system_user_connectivity.py
│   │   │   └── utils.py
│   │   ├── tests
│   │   │   ├── __init__.py
│   │   │   ├── test_system_user.py
│   │   │   └── tree.py
│   │   ├── urls
│   │   │   ├── api_urls.py
│   │   │   ├── __init__.py
│   │   │   └── views_urls.py
│   │   └── utils.py
│   ├── audits
│   │   ├── admin.py
│   │   ├── api.py
│   │   ├── apps.py
│   │   ├── filters.py
│   │   ├── hands.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   ├── 0002_ftplog_org_id.py
│   │   │   ├── 0003_auto_20180816_1652.py
│   │   │   ├── 0004_operatelog_passwordchangelog_userloginlog.py
│   │   │   ├── 0005_auto_20190228_1715.py
│   │   │   ├── 0006_auto_20190726_1753.py
│   │   │   ├── 0007_auto_20191202_1010.py
│   │   │   ├── 0008_auto_20200508_2105.py
│   │   │   ├── 0009_auto_20200624_1654.py
│   │   │   ├── 0010_auto_20200811_1122.py
│   │   │   ├── 0011_userloginlog_backend.py
│   │   │   ├── 0012_auto_20210414_1443.py
│   │   │   └── __init__.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── signals_handler.py
│   │   ├── tasks.py
│   │   ├── tests.py
│   │   ├── urls
│   │   │   ├── api_urls.py
│   │   │   ├── __init__.py
│   │   │   └── view_urls.py
│   │   └── utils.py
│   ├── authentication
│   │   ├── admin.py
│   │   ├── api
│   │   │   ├── access_key.py
│   │   │   ├── connection_token.py
│   │   │   ├── dingtalk.py
│   │   │   ├── __init__.py
│   │   │   ├── login_confirm.py
│   │   │   ├── mfa.py
│   │   │   ├── password.py
│   │   │   ├── sso.py
│   │   │   ├── token.py
│   │   │   └── wecom.py
│   │   ├── apps.py
│   │   ├── backends
│   │   │   ├── api.py
│   │   │   ├── cas
│   │   │   │   ├── backends.py
│   │   │   │   ├── callback.py
│   │   │   │   ├── __init__.py
│   │   │   │   ├── middleware.py
│   │   │   │   └── urls.py
│   │   │   ├── __init__.py
│   │   │   ├── ldap.py
│   │   │   ├── oidc
│   │   │   │   ├── __init__.py
│   │   │   │   └── middleware.py
│   │   │   ├── openid.py
│   │   │   ├── pubkey.py
│   │   │   └── radius.py
│   │   ├── const.py
│   │   ├── errors.py
│   │   ├── filters.py
│   │   ├── forms.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   ├── 0002_auto_20190729_1423.py
│   │   │   ├── 0003_loginconfirmsetting.py
│   │   │   ├── 0004_ssotoken.py
│   │   │   └── __init__.py
│   │   ├── mixins.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── signals_handlers.py
│   │   ├── signals.py
│   │   ├── tasks.py
│   │   ├── templates
│   │   │   └── authentication
│   │   │ ��     ├── _access_key_modal.html
│   │   │       ├── _captcha_field.html
│   │   │       ├── login.html
│   │   │       ├── login_otp.html
│   │   │       ├── login_wait_confirm.html
│   │   │       └── _mfa_confirm_modal.html
│   │   ├── tests.py
│   │   ├── urls
│   │   │   ├── api_urls.py
│   │   │   └── view_urls.py
│   │   ├── utils.py
│   │   └── views
│   │       ├── dingtalk.py
│   │       ├── __init__.py
│   │       ├── login.py
│   │       ├── mfa.py
│   │       ├── utils.py
│   │       └── wecom.py
│   ├── common
│   │   ├── api.py
│   │   ├── apps.py
│   │   ├── auth
│   │   │   ├── __init__.py
│   │   │   └── signature.py
│   │   ├── cache.py
│   │   ├── compat.py
│   │   ├── const
│   │   │   ├── choices.py
│   │   │   ├── http.py
│   │   │   ├── __init__.py
│   │   │   └── signals.py
│   │   ├── db
│   │   │   ├── aggregates.py
│   │   │   ├── __init__.py
│   │   │   ├── models.py
│   │   │   └── utils.py
│   │   ├── decorator.py
│   │   ├── drf
│   │   │   ├── api.py
│   │   │   ├── exc_handlers.py
│   │   │   ├── fields.py
│   │   │   ├── filters.py
│   │   │   ├── __init__.py
│   │   │   ├── metadata.py
│   │   │   ├── parsers
│   │   │   │   ├── base.py
│   │   │   │   ├── csv.py
│   │   │   │   ├── excel.py
│   │   │   │   └── __init__.py
│   │   │   ├── renders
│   │   │   │   ├── base.py
│   │   │   │   ├── csv.py
│   │   │   │   ├── excel.py
│   │   │   │   └── __init__.py
│   │   │   ├── routers.py
│   │   │   └── serializers.py
│   │   ├── exceptions.py
│   │   ├── fields
│   │   │   ├── __init__.py
│   │   │   └── model.py
│   │   ├── http.py
│   │   ├── __init__.py
│   │   ├── local.py
│   │   ├── management
│   │   │   └── commands
│   │   │       └── expire_caches.py
│   │   ├── message
│   │   │   ├── backends
│   │   │   │   ├── dingtalk
│   │   │   │   │   └── __init__.py
│   │   │   │   ├── exceptions.py
│   │   │   │   ├── __init__.py
│   │   │   │   ├── mixin.py
│   │   │   │   ├── utils.py
│   │   │   │   └── wecom
│   │   │   │       └── __init__.py
│   │   │   └── __init__.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   ├── 0002_auto_20180111_1407.py
│   │   │   ├── 0003_setting_category.py
│   │   │   ├── 0004_setting_encrypted.py
│   │   │   ├── 0005_auto_20190221_1902.py
│   │   │   ├── 0006_auto_20190304_1515.py
│   │   │   └── __init__.py
│   │   ├── mixins
│   │   │   ├── api.py
│   │   │   ├── __init__.py
│   │   │   ├── models.py
│   │   │   ├── serializers.py
│   │   │   └── views.py
│   │   ├── permissions.py
│   │   ├── README.md
│   │   ├── request_log.py
│   │   ├── signals_handlers.py
│   │   ├── signals.py
│   │   ├── struct.py
│   │   ├── tasks.py
│   │   ├── templatetags
│   │   │   ├── common_tags.py
│   │   │   └── __init__.py
│   │   ├── tests.py
│   │   ├── thread_pools.py
│   │   ├── tree.py
│   │   ├── urls
│   │   │   ├── api_urls.py
│   │   │   ├── __init__.py
│   │   │   └── view_urls.py
│   │   ├── utils
│   │   │   ├── common.py
│   │   │   ├── connection.py
│   │   │   ├── crypto.py
│   │   │   ├── django.py
│   │   │   ├── encode.py
│   │   │   ├── http.py
│   │   │   ├── __init__.py
│   │   │   ├── inspect.py
│   │   │   ├── ipip
│   │   │   │   ├── __init__.py
│   │   │   │   ├── ipipfree.ipdb
│   │   │   │   └── utils.py
│   │   │   ├── jumpserver.py
│   │   │   ├── lock.py
│   │   │   ├── random.py
│   │   │   ├── strings.py
│   │   │   └── timezone.py
│   │   ├── validators.py
│   │   └── views.py
│   ├── __init__.py
│   ├── jumpserver
│   │   ├── api.py
│   │   ├── asgi.py
│   │   ├── conf.py
│   │   ├── const.py
│   │   ├── context_processor.py
│   │   ├── __init__.py
│   │   ├── middleware.py
│   │   ├── rewriting
│   │   │   ├── __init__.py
│   │   │   └── session.py
│   │   ├── routing.py
│   │   ├── settings
│   │   │   ├── auth.py
│   │   │   ├── base.py
│   │   │   ├── custom.py
│   │   │   ├── __init__.py
│   │   │   ├── libs.py
│   │   │   ├── logging.py
│   │   │   └── _xpack.py
│   │   ├── urls.py
│   │   ├── utils.py
│   │   ├── views
│   │   │   ├── celery_flower.py
│   │   │   ├── error_views.py
│   │   │   ├── index.py
│   │   │   ├── __init__.py
│   │   │   ├── other.py
│   │   │   └── swagger.py
│   │   └── wsgi.py
│   ├── locale
│   │   └── zh
│   │       └── LC_MESSAGES
│   │           ├── djangojs.mo
│   │           ├── djangojs.po
│   │           ├── django.mo
│   │           └── django.po
│   ├── manage.py
│   ├── notifications
│   │   ├── api
│   │   │   ├── __init__.py
│   │   │   ├── notifications.py
│   │   │   └── site_msgs.py
│   │   ├── apps.py
│   │   ├── backends
│   │   │   ├── base.py
│   │   │   ├── dingtalk.py
│   │   │   ├── email.py
│   │   │   ├── __init__.py
│   │   │   ├── site_msg.py
│   │   │   └── wecom.py
│   │   ├── filters.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   └── __init__.py
│   │   ├── models
│   │   │   ├── __init__.py
│   │   │   ├── notification.py
│   │   │   └── site_msg.py
│   │   ├── notifications.py
│   │   ├── serializers
│   │   │   ├── __init__.py
│   │   │   ├── notifications.py
│   │   │   └── site_msgs.py
│   │   ├── signals_handler.py
│   │   ├── site_msg.py
│   │   ├── tests.py
│   │   ├── urls
│   │   │   ├── api_urls.py
│   │   │   ├── __init__.py
│   │   │   └── ws_urls.py
│   │   └── ws.py
│   ├── ops
│   │   ├── ansible
│   │   │   ├── callback.py
│   │   │   ├── display.py
│   │   │   ├── exceptions.py
│   │   │   ├── __init__.py
│   │   │   ├── inventory.py
│   │   │   ├── runner.py
│   │   │   ├── test_inventory.py
│   │   │   ├── test_runner.py
│   │   │   └── utils.py
│   │   ├── api
│   │   │   ├── adhoc.py
│   │   │   ├── celery.py
│   │   │   ├── command.py
│   │   │   └── __init__.py
│   │   ├── apps.py
│   │   ├── celery
│   │   │   ├── const.py
│   │   │   ├── decorator.py
│   │   │   ├── __init__.py
│   │   │   ├── logger.py
│   │   │   ├── signal_handler.py
│   │   │   └── utils.py
│   │   ├── hands.py
│   │   ├── __init__.py
│   │   ├── inventory.py
│   │   ├── management
│   │   │   └── commands
│   │   │       └── check_celery.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   ├── 0002_celerytask.py
│   │   │   ├── 0003_auto_20181207_1744.py
│   │   │   ├── 0004_adhoc_run_as.py
│   │   │   ├── 0005_auto_20181219_1807.py
│   │   │   ├── 0006_auto_20190318_1023.py
│   │   │   ├── 0007_auto_20190724_2002.py
│   │   │   ├── 0008_auto_20190919_2100.py
│   │   │   ├── 0009_auto_20191217_1713.py
│   │   │   ├── 0010_auto_20191217_1758.py
│   │   │   ├── 0011_auto_20200106_1534.py
│   │   │   ├── 0012_auto_20200108_1659.py
│   │   │   ├── 0013_auto_20200108_1706.py
│   │   │   ├── 0014_auto_20200108_1749.py
│   │   │   ├── 0015_auto_20200108_1809.py
│   │   │   ├── 0016_commandexecution_org_id.py
│   │   │   ├── 0017_auto_20200306_1747.py
│   │   │   ├── 0018_auto_20200509_1434.py
│   │   │   ├── 0019_adhocexecution_celery_task_id.py
│   │   │   ├── 0020_adhoc_run_system_user.py
│   │   │   └── __init__.py
│   │   ├── mixin.py
│   │   ├── models
│   │   │   ├── adhoc.py
│   │   │   ├── celery.py
│   │   │   ├── command.py
│   │   │   └── __init__.py
│   │   ├── notifications.py
│   │   ├── serializers
│   │   │   ├── adhoc.py
│   │   │   ├── celery.py
│   │   │   └── __init__.py
│   │   ├── tasks.py
│   │   ├── templates
│   │   │   └── ops
│   │   │       └── celery_task_log.html
│   │   ├── tests
│   │   │   ├── __init__.py
│   │   │   └── tests.py
│   │   ├── test_utils.py
│   │   ├── urls
│   │   │   ├── api_urls.py
│   │   │   ├── __init__.py
│   │   │   ├── view_urls.py
│   │   │   └── ws_urls.py
│   │   ├── utils.py
│   │   ├── views.py
│   │   └── ws.py
│   ├── orgs
│   │   ├── admin.py
│   │   ├── api.py
│   │   ├── apps.py
│   │   ├── caches.py
│   │   ├── context_processor.py
│   │   ├── filters.py
│   │   ├── hands.py
│   │   ├── __init__.py
│   │   ├── middleware.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   ├── 0002_auto_20180903_1132.py
│   │   │   ├── 0003_auto_20190916_1057.py
│   │   │   ├── 0004_organizationmember.py
│   │   │   ├── 0005_auto_20200721_1937.py
│   │   │   ├── 0006_auto_20200721_1937.py
│   │   │   ├── 0007_auto_20200728_1805.py
│   │   │   ├── 0008_auto_20200819_2041.py
│   │   │   ├── 0009_auto_20201023_1628.py
│   │   │   ├── 0010_auto_20210219_1241.py
│   │   │   └── __init__.py
│   │   ├── mixins
│   │   │   ├── api.py
│   │   │   ├── forms.py
│   │   │   ├── generics.py
│   │   │   ├── __init__.py
│   │   │   ├── models.py
│   │   │   └── serializers.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── signals_handler
│   │   │   ├── cache.py
│   │   │   ├── common.py
│   │   │   └── __init__.py
│   │   ├── tasks.py
│   │   ├── tests.py
│   │   ├── urls
│   │   │   ├── api_urls.py
│   │   │   ├── __init__.py
│   │   │   └── views_urls.py
│   │   └── utils.py
│   ├── perms
│   │   ├── api
│   │   │   ├── application
│   │   │   │   ├── application_permission.py
│   │   │   │   ├── application_permission_relation.py
│   │   │   │   ├── __init__.py
│   │   │   │   ├── user_group_permission.py
│   │   │   │   └── user_permission
│   │   │   │       ├── common.py
│   │   │   │       ├── __init__.py
│   │   │   │       └── user_permission_applications.py
│   │   │   ├── asset
│   │   │   │   ├── asset_permission.py
│   │   │   │   ├── asset_permission_relation.py
│   │   │   │   ├── __init__.py
│   │   │   │   ├── user_group_permission.py
│   │   │   │   └── user_permission
│   │   │   │       ├── common.py
│   │   │   │       ├── __init__.py
│   │   │   │       ├── mixin.py
│   │   │   │       ├── user_permission_assets
│   │   │   │       │   ├── __init__.py
│   │   │   │       │   ├── mixin.py
│   │   │   │       │   └── views.py
│   │   │   │       ├── user_permission_nodes.py
│   │   │   │       └── user_permission_nodes_with_assets.py
│   │   │   ├── base.py
│   │   │   ├── __init__.py
│   │   │   └── system_user_permission.py
│   │   ├── apps.py
│   │   ├── exceptions.py
│   │   ├── filters.py
│   │   ├── hands.py
│   │   ├── __init__.py
│   │   ├── locks.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   ├── 0002_auto_20171228_0025.py
│   │   │   ├── 0002_auto_20171228_0025_squashed_0009_auto_20180903_1132.py
│   │   │   ├── 0003_action.py
│   │   │   ├── 0004_assetpermission_actions.py
│   │   │   ├── 0005_auto_20190521_1619.py
│   │   │   ├── 0006_auto_20190628_1921.py
│   │   │   ├── 0007_remove_assetpermission_actions.py
│   │   │   ├── 0008_auto_20190911_1907.py
│   │   │   ├── 0009_remoteapppermission_system_users.py
│   │   │   ├── 0010_auto_20191218_1705.py
│   │   │   ├── 0011_auto_20200721_1739.py
│   │   │   ├── 0012_k8sapppermission.py
│   │   │   ├── 0013_rebuildusertreetask_usergrantedmappingnode.py
│   │   │   ├── 0014_build_users_perm_tree.py
│   │   │   ├── 0015_auto_20200929_1728.py
│   │   │   ├── 0016_applicationpermission.py
│   │   │   ├── 0017_auto_20210104_0435.py
│   │   │   ├── 0018_auto_20210208_1515.py
│   │   │   └── __init__.py
│   │   ├── mixins.py
│   │   ├── models
│   │   │   ├── application_permission.py
│   │   │   ├── asset_permission.py
│   │   │   ├── base.py
│   │   │   └── __init__.py
│   │   ├── pagination.py
│   │   ├── serializers
│   │   │   ├── application
│   │   │   │   ├── __init__.py
│   │   │   │   ├── permission.py
│   │   │   │   ├── permission_relation.py
│   │   │   │   └── user_permission.py
│   │   │   ├── asset
│   │   │   │   ├── __init__.py
│   │   │   │   ├── permission.py
│   │   │   │   ├── permission_relation.py
│   │   │   │   └── user_permission.py
│   │   │   ├── __init__.py
│   │   │   └── system_user_permission.py
│   │   ├── signals_handler
│   │   │   ├── common.py
│   │   │   ├── __init__.py
│   │   │   └── refresh_perms.py
│   │   ├── tasks.py
│   │   ├── tests.py
│   │   ├── urls
│   │   │   ├── api_urls.py
│   │   │   ├── application_permission.py
│   │   │   ├── asset_permission.py
│   │   │   ├── __init__.py
│   │   │   └── system_user_permission.py
│   │   └── utils
│   │       ├── application
│   │       │   ├── __init__.py
│   │       │   ├── permission.py
│   │       │   └── user_permission.py
│   │       ├── asset
│   │       │   ├── __init__.py
│   │       │   ├── permission.py
│   │       │   └── user_permission.py
│   │       └── __init__.py
│   ├── settings
│   │   ├── admin.py
│   │   ├── api
│   │   │   ├── common.py
│   │   │   ├── dingtalk.py
│   │   │   ├── __init__.py
│   │   │   ├── ldap.py
│   │   │   └── wecom.py
│   │   ├── apps.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   └── __init__.py
│   │   ├── models.py
│   │   ├── serializers
│   │   │   ├── email.py
│   │   │   ├── __init__.py
│   │   │   ├── ldap.py
│   │   │   ├── public.py
│   │   │   └── settings.py
│   │   ├── signals_handler.py
│   │   ├── tasks
│   │   │   ├── __init__.py
│   │   │   └── ldap.py
│   │   ├── tests.py
│   │   ├── urls
│   │   │   └── api_urls.py
│   │   └── utils
│   │       ├── common.py
│   │       ├── __init__.py
│   │       └── ldap.py
│   ├── static
│   │   ├── css
│   │   │   ├── animate.css
│   │   │   ├── bootstrap.min.css
│   │   │   ├── bootstrap-style.css
│   │   │   ├── colorbox.css
│   │   │   ├── font-awesome.min.css
│   │   │   ├── images
│   │   │   │   ├── jbox-button1.png
│   │   │   │   ├── jbox-close.gif
│   │   │   │   └── jbox-icons.png
│   │   │   ├── jumpserver.css
│   │   │   ├── login-style.css
│   │   │   ├── otp.css
│   │   │   ├── patterns
│   │   │   │   ├── congruent_pentagon.png
│   │   │   │   ├── header-profile.png
│   │   │   │   ├── header-profile-skin-1.png
│   │   │   │   ├── header-profile-skin-2.png
│   │   │   │   ├── header-profile-skin-3.png
│   │   │   │   ├── otis_redding.png
│   │   │   │   ├── shattered.png
│   │   │   │   └── triangular.png
│   │   │   ├── plugins
│   │   │   │   ├── awesome-bootstrap-checkbox
│   │   │   │   │   └── awesome-bootstrap-checkbox.css
│   │   │   │   ├── bootstrap.min.css
│   │   │   │   ├── codemirror
│   │   │   │   │   ├── ambiance.css
│   │   │   │   │   └── codemirror.css
│   │   │   │   ├── cropper
│   │   │   │   │   └── cropper.min.css
│   │   │   │   ├── datatables
│   │   │   │   │   ├── datatables.min.css
│   │   │   │   │   └── datatables.min.css.bak
│   │   │   │   ├── datepicker
│   │   │   │   │   └── datepicker3.css
│   │   │   │   ├── daterangepicker
│   │   │   │   │   └── daterangepicker.css
│   │   │   │   ├── dropzone
│   │   │   │   │   ├── basic.css
│   │   │   │   │   └── dropzone.css
│   │   │   │   ├── footable
│   │ �� │   │   │   ├── fonts
│   │   │   │   │   │   ├── footable.eot
│   │   │   │   │   │   ├── footable.svg
│   │   │   │   │   │   ├── footable.ttf
│   │   │   │   │   │   └── footable.woff
│   │   │   │   │   └── footable.core.css
│   │   │   │   ├── fullcalendar
│   │   │   │   │   ├── fullcalendar.css
│   │   │   │   │   └── fullcalendar.print.css
│   │   │   │   ├── iCheck
│   │   │   │   │   ├── custom.css
│   │   │   │   │   ├── green@2x.png
│   │   │   │   │   └── green.png
│   │   │   │   ├── images
│   │   │   │   │   ├── bootstrap-colorpicker
│   │   │   │   │   │   ├── alpha-horizontal.png
│   │   │   │   │   │   ├── alpha.png
│   │   │   │   │   │   ├── hue-horizontal.png
│   │   │   │   │   │   ├── hue.png
│   │   │   │   │   │   └── saturation.png
│   │   │   │   │   ├── sort_asc.png
│   │   │   │   │   ├── sort_desc.png
│   │   │   │   │   ├── sort.png
│   │   │   │   │   ├── spritemap@2x.png
│   │   │   │   │   ├── spritemap.png
│   │   │   │   │   ├── sprite-skin-flat2.png
│   │   │   │   │   ├── sprite-skin-flat.png
│   │   │   │   │   ├── sprite-skin-nice.png
│   │   │   │   │   └── sprite-skin-simple.png
│   │   │   │   ├── inputTags.css
│   │   │   │   ├── jstree
│   │   │   │   │   ├── 32px.png
│   │   │   │   │   ├── 39px.png
│   │   │   │   │   ├── 40px.png
│   │   │   │   │   ├── style.css
│   │   │   │   │   ├── style.min.css
│   │   │   │   │   └── throbber.gif
│   │   │   │   ├── ladda
│   │   │   │   │   ├── ladda.min.css
│   │   │   │   │   └── ladda-themeless.min.css
│   │   │   │   ├── layer
│   │   │   │   │   ├── default
│   │   │   │   │   │   ├── icon-ext.png
│   │   │   │   │   │   ├── icon.png
│   │   │   │   │   │   ├── loading-0.gif
│   │   │   │   │   │   ├── loading-1.gif
│   │   │   │   │   │   └── loading-2.gif
│   │   │   │   │   └── layer.css
│   │   │   │   ├── select2
│   │   │   │   │   └── select2.min.css
│   │   │   │   ├── steps
│   │   │   │   │   └── jquery.steps.css
│   │   │   │   ├── sweetalert
│   │   │   │   │   └── sweetalert.css
│   │   │   │   ├── toastr
│   │   │   │   │   └── toastr.min.css
│   │   │   │   ├── vaildator
│   │   │   │   │   ├── images
│   │   │   │   │   │   ├── loading.gif
│   │   │   │   │   │   ├── validator_default.png
│   │   │   │   │   │   └── validator_simple.png
│   │   │   │   │   └── jquery.validator.css
│   │   │   │   └── ztree
│   │   │   ���       ├── awesomeStyle
│   │   │   │       │   ├── awesome.css
│   │   │   │       │   ├── awesome.less
│   │   │   │       │   ├── fa.css
│   │   │   │       │   ├── fa.less
│   │   │   │       │   └── img
│   │   │   │       │       └── loading.gif
│   │   │   │       ├── demo.css
│   │   │   │       ├── metroStyle
│   │   │   │       │   ├── img
│   │   │   │       │   │   ├── line_conn.png
│   │   │   │       │   │   ├── loading.gif
│   │   │   │       │   │   ├── metro.gif
│   │   │   │       │   │   └── metro.png
│   │   │   │       │   └── metroStyle.css
│   │   │   │       └── ztreestyle
│   │   │   │           ├── img
│   │   │   │           │   ├── diy
│   │   │   │           │   │   ├── 1_close.png
│   │   │   │           │   │   ├── 1_open.png
│   │   │   │           │   │   ├── 2.png
│   │   │   │           │   │   ├── 3.png
│   │   │   │           │   │   ├── 4.png
│   │   │   │           │   │   ├── 5.png
│   │   │   │           │   │   ├── 6.png
│   │   │   │           │   │   ├── 7.png
│   │   │   │           │   │   ├── 8.png
│   │   │   │           │   │   └── 9.png
│   │   │   │           │   ├── line_conn.gif
│   │   │   │           │   ├── loading.gif
│   │   │   │           │   ├── zTreeStandard.gif
│   │   │   │           │   └── zTreeStandard.png
│   │   │   │           └── ztreestyle.css
│   │   │   └── style.css
│   │   ├── fonts
│   │   │   ├── FontAwesome.otf
│   │   │   ├── fontawesome-webfont.eot
│   │   │   ├── fontawesome-webfont.svg
│   │   │   ├── fontawesome-webfont.ttf
│   │   │   ├── fontawesome-webfont.woff
│   │   │   ├── fontawesome-webfont.woff2
│   │   │   ├── font_otp
│   │   │   │   ├── iconfont.css
│   │   │   │   ├── iconfont.eot
│   │   │   │   ├── iconfont.js
│   │   │   │   ├── iconfont.svg
│   │   │   │   ├── iconfont.ttf
│   │   │   │   └── iconfont.woff
│   │   │   ├── glyphicons-halflings-regular.eot
│   │   │   ├── glyphicons-halflings-regular.svg
│   │   │   ├── glyphicons-halflings-regular.ttf
│   │   │   ├── glyphicons-halflings-regular.woff
│   │   │   └── glyphicons-halflings-regular.woff2
│   │   ├── img
│   │   │   ├── authenticator_android.png
│   │   │   ├── authenticator_iphone.png
│   │   │   ├── avatar
│   │   │   │   ├── admin.png
│   │   │   │   └── user.png
│   │   │   ├── facio.ico
│   │   │   ├── header-profile.png
│   │   │   ├── login_cas_logo.png
│   │   │   ├── login_dingtalk_logo.png
│   │   │   ├── login_image.jpg
│   │   │   ├── login_wecom_logo.png
│   │   │   ├── logo.png
│   │   │   ├── logo_text.png
│   │   │   ├── otp_auth.png
│   │   │   └── root.png
│   │   └── js
│   │       ├── angular.min.js
│   │       ├── angular-route.min.js
│   │       ├── bootstrap-dialog.js
│   │       ├── bootstrap.min.js
│   │       ├── inspinia.js
│   │       ├── jquery-2.1.1.js
│   │       ├── jquery-3.1.1.min.js
│   │       ├── jquery.colorbox.js
│   │       ├── jquery.form.min.js
│   │       ├── jquery.shiftcheckbox.js
│   │       ├── jquery-ui-1.10.4.min.js
│   │       ├── jquery-ui-1.12.1.js
│   │       ├── jquery-ui.custom.min.js
│   │       ├── jumpserver.js
│   │       ├── mindmup-editabletable.js
│   │       ├── plugins
│   │       │   ├── clipboard
│   │       │   │   └── clipboard.min.js
│   │       │   ├── codemirror
│   │       │   │   ├── codemirror.js
│   │       │   │   └── mode
│   │       │   │       ├── index.html
│   │       │   │       ├── meta.js
│   │       │   │       └── shell
│   │       │   │           ├── index.html
│   │       │   │           ├── shell.js
│   │       │   │           └── test.js
│   │       │   ├── cropper
│   │       │   │   └── cropper.min.js
│   │       │   ├── datatables
│   │       │   │   ├── datatables.min.js
│   │       │   │   ├── i18n
│   │       │   │   │   ├── English.lang
│   │       │   │   │   └── zh-hans.json
│   │       │   │   └── pdfmake.min.js.map
│   │       │   ├── datepicker
│   │       │   │   ├── bootstrap-datepicker.js
│   │       │   │   └── bootstrap-datepicker.zh-CN.min.js
│   │       │   ├── daterangepicker
│   │       │   │   ├── daterangepicker.min.js
│   │       │   │   └── moment.min.js
│   │       │   ├── demo
│   │       │   │   └── peity-demo.js
│   │       │   ├── dropzone
│   │       │   │   └── dropzone.js
│   │       │   ├── echarts
│   │       │   │   ├── chart
│   │       │   │   │   ├── bar.js
│   │       │   │   │   ├── chord.js
│   │       │   │   │   ├── eventRiver.js
│   │       │   │   │   ├── force.js
│   │       │   │   │   ├── funnel.js
│   │       │   │   │   ├── gauge.js
│   │       │   │   │   ├── heatmap.js
│   │       │   │   │   ├── k.js
│   │       │   │   │   ├── line.js
│   │       │   │   │   ├── map.js
│   │       │   │   │   ├── pie.js
│   │       │   │   │   ├── radar.js
│   │       │   │   │   ├── scatter.js
│   │       │   │   │   ├── tree.js
│   │       │   │   │   ├── treemap.js
│   │       │   │   │   ├── venn.js
│   │       │   │   │   └── wordCloud.js
│   │       │   │   ├── echarts-all.js
│   │       │   │   ├── echarts.js
│   │       │   │   └── echarts.min.js
│   │       │   ├── footable
│   │       │   │   ├── footable.all.min.js
│   │       │   │   └── footable.min.js
│   │       │   ├── fullcalendar
│   │       │   │   ├── fullcalendar.min.js
│   │       │   │   └── moment.min.js
│   │       │   ├── highcharts
│   │       │   │   ├── adapters
│   │       │   │   │   ├── standalone-framework.js
│   │       │   │   │   └── standalone-framework.src.js
│   │       │   │   ├── highcharts-3d.js
│   │       │   │   ├── highcharts-3d.src.js
│   │       │   │   ├── highcharts-all.js
│   │       │   │   ├── highcharts.js
│   │       │   │   ├── highcharts-more.js
│   │       │   │   ├── highcharts-more.src.js
│   │       │   │   ├── highcharts.src.js
│   │       │   │   ├── modules
│   │       │   │   │   ├── canvas-tools.js
│   │       │   │   │   ├── canvas-tools.src.js
│   │       │   │   │   ├── data.js
│   │       │   │   │   ├── data.src.js
│   │       │   │   │   ├── drilldown.js
│   │       │   │   │   ├── drilldown.src.js
│   │       │   │   │   ├── exporting.js
│   │       │   │   │   ├── exporting.src.js
│   │       │   │   │   ├── funnel.js
│   │       │   │   │   ├── funnel.src.js
│   │       │   │   │   ├── heatmap.js
│   │       │   │   │   ├── heatmap.src.js
│   │       │   │   │   ├── no-data-to-display.js
│   │       │   │   │   ├── no-data-to-display.src.js
│   │       │   │   │   ├── solid-gauge.js
│   │       │   │   │   └── solid-gauge.src.js
│   │       │   │   └── themes
│   │       │   │       ├── dark-blue.js
│   │       │   │       ├── dark-green.js
│   │       │   │       ├── dark-unica.js
│   │       │   │       ├── gray.js
│   │       │   │       ├── grid.js
│   │       │   │       ├── grid-light.js
│   │       │   │       ├── sand-signika.js
│   │       │   │       └── skies.js
│   │       │   ├── iCheck
│   │       │   │   └── icheck.min.js
│   │       │   ├── inputTags.jquery.min.js
│   │       │   ├── jsencrypt
│   │       │   │   └── jsencrypt.min.js
│   │       │   ├── jstree
│   │       │   │   └── jstree.min.js
│   │       │   ├── ladda
│   │       │   │   ├── ladda.jquery.min.js
│   │       │   │   ├── ladda.min.js
│   │       │   │   └── spin.min.js
│   │       │   ├── layer
│   │       │   │   ├── layer.js
│   │       │   │   └── skin
│   │       │   │       ├── default
│   │       │   │       │   ├── icon-ext.png
│   │       │   │       │   ├── icon.png
│   │       │   │       │   ├── loading-0.gif
│   │       │   │       │   ├── loading-1.gif
│   │       │   │       │   └── loading-2.gif
│   │       │   │       └── layer.css
│   │       │   ├── magnific
│   │       │   │   ���── jquery.magnific-popup.min.js
│   │       │   ├── metisMenu
│   │       │   │   └── jquery.metisMenu.js
│   │       │   ├── moment
│   │       │   │   └── moment.min.js
│   │       │   ├── pace
│   │       │   │   └── pace.min.js
│   │       │   ├── peity
│   │       │   │   └── jquery.peity.min.js
│   │       │   ├── qrcode
│   │       │   │   └── qrcode.min.js
│   │       │   ├── select2
│   │       │   │   ├── i18n
│   │       │   │   │   └── zh-CN.js
│   │       │   │   └── select2.full.min.js
│   │       │   ├── slimscroll
│   │       │   │   ├── jquery.slimscroll.js
│   │       │   │   └── jquery.slimscroll.min.js
│   │       │   ├── steps
│   │       │   │   └── jquery.steps.min.js
│   │       │   ├── sweetalert
│   │       │   │   └── sweetalert.min.js
│   │       │   ├── toastr
│   │       │   │   └── toastr.min.js
│   │       │   ├── validate
│   │       │   │   └── jquery.validate.min.js
│   │       │   ├── validator
│   │       │   │   ├── images
│   │       │   │   │   ├── loading.gif
│   │       │   │   │   ├── validator_default.png
│   │       │   │   │   └── validator_simple.png
│   │       │   │   ├── jquery.validator.js
│   │       │   │   └── zh_CN.js
│   │       │   ├── xterm
│   │       │   │   ├── addons
│   │       │   │   │   ├── attach
│   │       │   │   │   │   ├── attach.js
│   │       │   │   │   │   └── attach.js.map
│   │       │   │   │   ├── fit
│   │       │   │   │   │   ├── fit.js
│   │       │   │   │   │   └── fit.js.map
│   │       │   │   │   ├── fullscreen
│   │       │   │   │   │   ├── fullscreen.css
│   │       │   │   │   │   ├── fullscreen.js
│   │       │   │   │   │   └── fullscreen.js.map
│   │       │   │   │   ├── search
│   │       │   │   │   │   ├── search.js
│   │       │   │   │   │   └── search.js.map
│   │       │   │   │   ├── terminado
│   │       │   │   │   │   ├── terminado.js
│   │       │   │   │   │   └── terminado.js.map
│   │       │   │   │   ├── webLinks
│   │       │   │   │   │   ├── webLinks.js
│   │       │   │   │   │   └── webLinks.js.map
│   │       │   │   │   ├── winptyCompat
│   │       │   │   │   │   ├── winptyCompat.js
│   │       │   │   │   │   └── winptyCompat.js.map
│   │       │   │   │   └── zmodem
│   │       │   │   │       ├── zmodem.js
│   │       │   │   │       └── zmodem.js.map
│   │       │   │   ├── xterm.css
│   │       │   │   ├── xterm.js
│   │       │   │   └── xterm.js.map
│   │       │   └── ztree
│   │       │       ├── jquery-1.4.4.min.js
│   │       │       ├── jquery.ztree.all.min.js
│   │       │       └── jquery.ztree.exhide.min.js
│   │       ├── pwstrength-bootstrap.js
│   │       ├── record.js
│   │       ├── socket.io.js
│   │       └── term.js
│   ├── templates
│   │   ├── 404.html
│   │   ├── 500.html
│   │   ├── _base_asset_tree_list.html
│   │   ├── _base_create_update.html
│   │   ├── base.html
│   │   ├── _base_list.html
│   │   ├── _base_only_content.html
│   │   ├── _build.html
│   │   ├── _copyright.html
│   │   ├── _csv_import_export.html
│   │   ├── _csv_import_modal.html
│   │   ├── _csv_update_modal.html
│   │   ├── delete_confirm.html
│   │   ├── _filter_dropdown.html
│   │   ├── flash_message_standalone.html
│   │   ├── _footer.html
│   │   ├── _foot_js.html
│   │   ├── _head_css_js.html
│   │   ├── _header_bar.html
│   │   ├── index.html
│   │   ├── _left_side_bar.html
│   │   ├── _message.html
│   │   ├── _modal.html
│   │   ├── _nav.html
│   │   ├── _nav_user.html
│   │   ├── _pagination.html
│   │   ├── rest_framework
│   │   │   └── base.html
│   │   ├── _user_profile.html
│   │   └── _without_nav_base.html
│   ├── terminal
│   │   ├── admin.py
│   │   ├── api
│   │   │   ├── command.py
│   │   │   ├── __init__.py
│   │   │   ├── session.py
│   │   │   ├── status.py
│   │   │   ├── storage.py
│   │   │   ├── task.py
│   │   │   └── terminal.py
│   │   ├── apps.py
│   │   ├── backends
│   │   │   ├── command
│   │   │   │   ├── base.py
│   │   │   │   ├── db.py
│   │   │   │   ├── es.py
│   │   │   │   ├── __init__.py
│   │   │   │   ├── models.py
│   │   │   │   ├── multi.py
│   │   │   │   └── serializers.py
│   │   │   ├── __init__.py
│   │   │   └── replay
│   │   │       └── __init__.py
│   │   ├── const.py
│   │   ├── exceptions.py
│   │   ├── filters.py
│   │   ├── hands.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   ├── 0002_auto_20171228_0025.py
│   │   │   ├── 0002_auto_20171228_0025_squashed_0009_auto_20180326_0957.py
│   │   │   ├── 0003_auto_20171230_0308.py
│   │   │   ├── 0004_session_remote_addr.py
│   │   │   ├── 0005_auto_20180122_1154.py
│   │   │   ├── 0006_auto_20180123_1037.py
│   │   │   ├── 0007_session_date_last_active.py
│   │   │   ├── 0008_auto_20180307_1603.py
│   │   │   ├── 0009_auto_20180326_0957.py
│   │   │   ├── 0010_auto_20180423_1140.py
│   │   │   ├── 0011_auto_20180807_1116.py
│   │   │   ├── 0012_auto_20180816_1652.py
│   │   │   ├── 0013_auto_20181123_1113.py
│   │   │   ├── 0014_auto_20181226_1441.py
│   │   │   ├── 0015_auto_20190923_1529.py
│   │   │   ├── 0016_commandstorage_replaystorage.py
│   │   │   ├── 0017_auto_20191125_0931.py
│   │   │   ├── 0018_auto_20191202_1010.py
│   │   │   ├── 0019_auto_20191206_1000.py
│   │   │   ├── 0020_auto_20191218_1721.py
│   │   │   ├── 0021_auto_20200213_1316.py
│   │   │   ├── 0022_session_is_success.py
│   │   │   ├── 0023_command_risk_level.py
│   │   │   ├── 0024_auto_20200715_1713.py
│   │   │   ├── 0025_auto_20200810_1735.py
│   │   │   ├── 0026_auto_20201027_1905.py
│   │   │   ├── 0027_auto_20201102_1651.py
│   │   │   ├── 0028_auto_20201110_1918.py
│   │   │   ├── 0029_auto_20201116_1757.py
│   │   │   ├── 0030_terminal_type.py
│   │   │   ├── 0031_auto_20210113_1356.py
│   │   │   ├── 0032_auto_20210302_1853.py
│   │   │   ├── 0033_auto_20210324_1008.py
│   │   │   ├── 0034_auto_20210406_1434.py
│   │   │   ├── 0035_auto_20210517_1448.py
│   │   │   ├── 0036_auto_20210604_1124.py
│   │   │   ├── 0037_auto_20210623_1748.py
│   │   │   └── __init__.py
│   │   ├── models
│   │   │   ├── command.py
│   │   │   ├── __init__.py
│   │   │   ├── session.py
│   │   │   ├── status.py
│   │   │   ├── storage.py
│   │   │   ├── task.py
│   │   │   └── terminal.py
│   │   ├── notifications.py
│   │   ├── serializers
│   │   │   ├── command.py
│   │   │   ├── __init__.py
│   │   │   ├── session.py
│   │   │   ├── storage.py
│   │   │   └── terminal.py
│   │   ├── signals_handler.py
│   │   ├── tasks.py
│   │   ├── tests.py
│   │   ├── urls
│   │   │   ├── api_urls.py
│   │   │   └── __init__.py
│   │   └── utils.py
│   ├── tickets
│   │   ├── admin.py
│   │   ├── api
│   │   │   ├── assignee.py
│   │   │   ├── comment.py
│   │   │   ├── common.py
│   │   │   ├── __init__.py
│   │   │   └── ticket.py
│   │   ├── apps.py
│   │   ├── const.py
│   │   ├── handler
│   │   │   ├── apply_application.py
│   │   │   ├── apply_asset.py
│   │   │   ├── base.py
│   │   │   ├── command_confirm.py
│   │   │   ├── general.py
│   │   │   ├── __init__.py
│   │   │   ├── login_asset_confirm.py
│   │   │   └── login_confirm.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   ├── 0002_auto_20200728_1146.py
│   │   │   ├── 0003_auto_20200804_1551.py
│   │   │   ├── 0004_ticket_comment.py
│   │   │   ├── 0005_ticket_meta_confirmed_system_users.py
│   │   │   ├── 0006_auto_20201023_1628.py
│   │   │   ├── 0007_auto_20201224_1821.py
│   │   │   ├── 0008_auto_20210311_1113.py
│   │   │   ├── 0009_auto_20210426_1720.py
│   │   │   └── __init__.py
│   │   ├── models
│   │   │   ├── comment.py
│   │   │   ├── __init__.py
│   │   │   └── ticket.py
│   │   ├── permissions
│   │   │   ├── comment.py
│   │   │   ├── __init__.py
│   │   │   └── ticket.py
│   │   ├── serializers
│   │   │   ├── assignee.py
│   │   │   ├── comment.py
│   │   │   ├── __init__.py
│   │   │   └── ticket
│   │   │       ├── __init__.py
│   │   │       ├── meta
│   │   │       │   ├── __init__.py
│   │   │       │   ├── meta.py
│   │   │       │   └── ticket_type
│   │   │       │       ├── apply_application.py
│   │   │       │       ├── apply_asset.py
│   │   │       │       ├── command_confirm.py
│   │   │       │       ├── common.py
│   │   │       │       ├── __init__.py
│   │   │       │       ├── login_asset_confirm.py
│   │   │       │       └── login_confirm.py
│   │   │       └── ticket.py
│   │   ├── signals_handler
│   │   │   ├── comment.py
│   │   │   ├── __init__.py
│   │   │   └── ticket.py
│   │   ├── signals.py
│   │   ├── tests
│   │   │   └── __init__.py
│   │   ├── urls
│   │   │   ├── api_urls.py
│   │   │   └── __init__.py
│   │   └── utils.py
│   └── users
│       ├── api
│       │   ├── group.py
│       │   ├── __init__.py
│       │   ├── mixins.py
│       │   ├── profile.py
│       │   ├── relation.py
│       │   ├── service_account.py
│       │   └── user.py
│       ├── apps.py
│       ├── exceptions.py
│       ├── filters.py
│       ├── forms
│       │   ├── __init__.py
│       │   └── profile.py
│       ├── hands.py
│       ├── __init__.py
│       ├── migrations
│       │   ├── 0001_initial.py
│       │   ├── 0002_auto_20171225_1157.py
│       │   ├── 0002_auto_20171225_1157_squashed_0019_auto_20190304_1459.py
│       │   ├── 0003_auto_20180101_0046.py
│       │   ├── 0004_auto_20180125_1218.py
│       │   ├── 0005_auto_20180306_1804.py
│       │   ├── 0006_auto_20180411_1135.py
│       │   ├── 0007_auto_20180419_1036.py
│       │   ├── 0008_auto_20180425_1516.py
│       │   ├── 0009_auto_20180517_1537.py
│       │   ├── 0010_auto_20180606_1505.py
│       │   ├── 0011_user_source.py
│       │   ├── 0012_auto_20180710_1641.py
│       │   ├── 0013_auto_20180807_1116.py
│       │   ├── 0014_auto_20180816_1652.py
│       │   ├── 0015_auto_20181105_1112.py
│       │   ├── 0016_auto_20181109_1505.py
│       │   ├── 0017_auto_20181123_1113.py
│       │   ├── 0018_auto_20190107_1912.py
│       │   ├── 0019_auto_20190304_1459.py
│       │   ├── 0020_auto_20190612_1825.py
│       │   ├── 0021_auto_20190625_1104.py
│       │   ├── 0022_auto_20190625_1105.py
│       │   ├── 0023_auto_20190724_1525.py
│       │   ├── 0024_auto_20191118_1612.py
│       │   ├── 0025_auto_20200206_1216.py
│       │   ├── 0026_auto_20200508_2105.py
│       │   ├── 0027_auto_20200616_1503.py
│       │   ├── 0028_auto_20200728_1805.py
│       │   ├── 0029_auto_20200814_1650.py
│       │   ├── 0030_auto_20200819_2041.py
│       │   ├── 0031_auto_20201118_1801.py
│       │   ├── 0032_userpasswordhistory.py
│       │   ├── 0033_user_need_update_password.py
│       │   ├── 0034_auto_20210506_1448.py
│       │   ├── 0035_auto_20210526_1100.py
│       │   └── __init__.py
│       ├── models
│       │   ├── group.py
│       │   ├── __init__.py
│       │   ├── user.py
│       │   └── utils.py
│       ├── permissions.py
│       ├── serializers
│       │   ├── group.py
│       │   ├── __init__.py
│       │   ├── profile.py
│       │   ├── realtion.py
│       │   └── user.py
│       ├── signals_handler.py
│       ├── signals.py
│       ├── tasks.py
│       ├── templates
│       │   └── users
│       │       ├── _base_otp.html
│       │       ├── _base_user_detail.html
│       │       ├── first_login_done.html
│       │       ├── first_login.html
│       │       ├── forgot_password.html
│       │       ├── _granted_assets.html
│       │       ├── reset_password.html
│       │       ├── _select_user_modal.html
│       │       ├── user_asset_permission.html
│       │       ├── user_database_app_permission.html
│       │       ├── _user_detail_nav_header.html
│       │       ├── user_otp_check_password.html
│       │       ├── user_otp_enable_bind.html
│       │       ├── user_otp_enable_install_app.html
│       │       ├── user_password_update.html
│       │       ├── user_password_verify.html
│       │       ├── _user_update_pk_modal.html
│       │       └── user_verify_mfa.html
│       ├── tests
│       │   └── __init__.py
│       ├── urls
│       │   ├── api_urls.py
│       │   └── __init__.py
│       ├── utils.py
│       └── views
│           ├── __init__.py
│           └── profile
│               ├── __init__.py
│               ├── mfa.py
│               ├── otp.py
│               ├── password.py
│               ├── pubkey.py
│               └── reset.py
├── config_example.yml
├── data
│   ├── caution.txt
│   └── media
├── Dockerfile
├── docs
│   └── README.md
├── entrypoint.sh
├── jms
├── LICENSE
├── logs
├── README_EN.md
├── README.md
├── requirements
│   ├── alpine_requirements.txt
│   ├── deb_buster_requirements.txt
│   ├── deb_requirements.txt
│   ├── issues.txt
│   ├── mac_requirements.txt
│   ├── requirements.txt
│   └── rpm_requirements.txt
├── run_server.py
├── tmp
├── utils
│   ├── 1.4.4_to_1.4.5_migrations.sh
│   ├── 2018_04_11_migrate_permissions.sh
│   ├── 2018_07_15_set_win_protocol_to_ssh.sh
│   ├── backup_db.sh
│   ├── build_docker.sh
│   ├── build.sh
│   ├── clean_duplicate_user_groups.py
│   ├── clean_migrations.sh
│   ├── create_assets_user
│   │   ├── admin_users.txt
│   │   ├── bulk_create_user.py
│   │   ├── system_users.txt
│   │   └── \344\275\277\347\224\250\350\257\264\346\230\216.txt
│   ├── create_test_data.py
│   ├── disable_ldap_auth.sh
│   ├── disable_user_mfa.sh
│   ├── django_migrations.sql
│   ├── example_api.py
│   ├── export_fake_data.sh
│   ├── generate_fake_data
│   │   ├── generate.py
│   │   ├── __init__.py
│   │   └── resources
│   │       ├── assets.py
│   │       ├── base.py
│   │       ├── __init__.py
│   │       ├── perms.py
│   │       ├── system.py
│   │       └── users.py
│   ├── get_no_parent_nodes.py
│   ├── load_fake_data.sh
│   ├── make_migrations.sh
│   ├── migrate_unorg_users_to_default_org.sh
│   ├── redis.conf
│   ├── reupload_guacamole_replays.py
│   ├── start_celery_beat.py
│   ├── sync_node.py
│   ├── unblock_all_user.sh
│   └── upgrade.sh
└── Vagrantfile

255 directories, 1258 files
[root@master-61 jumpserver-v2.12.0]#
```

### 5 .安装rpm包

```
#1.jumpserver源码，给你提供了一堆rpm包，需要你去安装运行即可
/opt/jumpserver-v2.12.0/requirements
[root@master-61 jumpserver-v2.12.0]# ls
apps                Dockerfile     jms      README_EN.md  run_server.py  Vagrantfile
config_example.yml  docs           LICENSE  README.md     tmp
data                entrypoint.sh  logs     requirements  utils
[root@master-61 jumpserver-v2.12.0]# cd requirements/
[root@master-61 requirements]# ls
alpine_requirements.txt      deb_requirements.txt  mac_requirements.txt  rpm_requirements.txt
deb_buster_requirements.txt  issues.txt            requirements.txt
[root@master-61 requirements]


#
yum -y install $(cat /opt/jumpserver-v2.12.0/requirements/rpm_requirements.txt)
```

### 6.安装python3.6开发环境

```
#维护python产品的
1.python2版本，已经不维护了。老企业还在使用（Linux系统默认很多工具用的都是python2，如yum工具）
[root@master-61 jumpserver-v2.12.0]# which python
/usr/bin/python
[root@master-61 jumpserver-v2.12.0]# ll /usr/bin/python
lrwxrwxrwx. 1 root root 7 5月  27 09:46 /usr/bin/python -> python2
[root@master-61 jumpserver-v2.12.0]# python -V
Python 2.7.5

2.python3，主流版本

3.注意，同时保留python2和python3，不用乱改
#添加到PATH，让两个版本共存
#python语言是c语言开发而来的
#1.安装python3的编译环境
yum -y install gcc patch libffi-devel python-devel zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel 

# 2.下载python源码，编译安装
https://www.python.org/ftp/python/3.6.9/Python-3.6.9.tgz
tar -zxf Python-3.6.9.tgz
cd tar Python-3.6.9
./configure --prefix=/opt/python369
make && make install

#注意编译过程，没有error
checking for the getrandom() function... no
configure: creating ./config.status
config.status: creating Makefile.pre
config.status: creating Modules/Setup.config
config.status: creating Misc/python.pc
config.status: creating Misc/python-config.sh
config.status: creating Modules/ld_so_aix
config.status: creating pyconfig.h
creating Modules/Setup
creating Modules/Setup.local
creating Makefile


If you want a release build with all stable optimizations active (PGO, etc),
please run ./configure --enable-optimizations


[root@master-61 Python-3.6.9]# 


#make && make install
看到下面的日志，python3安装完成
Looking in links: /tmp/tmpec1lvfc2
Collecting setuptools
Collecting pip
Installing collected packages: setuptools, pip
Successfully installed pip-18.1 setuptools-40.6.2


#3.检查python3的环境，添加到PATH变量中
[root@master-61 Python-3.6.9]# cd /opt/python369/
[root@master-61 python369]# ls
bin  include  lib  share
[root@master-61 python369]# cd bin/
[root@master-61 bin]# ls
2to3              idle3    pip3.6    python3           python3.6m         pyvenv
2to3-3.6          idle3.6  pydoc3    python3.6         python3.6m-config  pyvenv-3.6
easy_install-3.6  pip3     pydoc3.6  python3.6-config  python3-config
[root@master-61 bin]# pwd
/opt/python369/bin
[root@master-61 bin]# vim /etc/profile
[root@master-61 bin]# tail -2 /etc/profile
#添加python3的PATH环境变量
export PATH=$PATH:/opt/python369/bin
[root@master-61 bin]#  source /etc/profile


# 4.确保如下命令可以正确执行
[root@master-61 bin]# python   #tab键补齐
python             python2.7-config   python3.6          python3.6m-config  
python2            python2-config     python3.6-config   python3-config     
python2.7          python3            python3.6m         python-config  

#这是python3的解释器，用于执行*.py代码的
[root@master-61 bin]# python3 -V
Python 3.6.9
[root@master-61 ~]# python3 -V
Python 3.6.9

#管理python程序的吗模块依赖程序包的，类似Yum是给CentOS安装rpm包依赖的，pip3是给python项目安装项目所需要的依赖
[root@master-61 ~]# pip3 -V
pip 18.1 from /opt/python369/lib/python3.6/site-packages/pip (python 3.6)
[root@master-61 ~]# 
```

### 7.python3的虚拟环境安装

```
#原理：就是修改PATH变量
#基于python3命令，指定模块venv这个模块功能，安装一个解释器路径到/opt/venv_pyt3
cd /opt && python3 -m venv /opt/venv_py3


#python3的第一个物理解释器，路径如下
root@master-61 opt]# ls -d /opt/python369/bin/
/opt/python369/bin/

#创建的虚拟环境，路径如下，用于单独管理一个python项目
[root@master-61 opt]# ls -d /opt/venv_py3/bin/
/opt/venv_py3/bin/
[root@master-61 opt]# 


#路径下面的内容
[root@master-61 opt]# ls -l /opt/python369/bin/
总用量 24864
lrwxrwxrwx. 1 root root        8 5月  27 11:09 2to3 -> 2to3-3.6
-rwxr-xr-x. 1 root root      105 5月  27 11:09 2to3-3.6
-rwxr-xr-x. 1 root root      246 5月  27 11:09 easy_install-3.6
lrwxrwxrwx. 1 root root        7 5月  27 11:09 idle3 -> idle3.6
-rwxr-xr-x. 1 root root      103 5月  27 11:09 idle3.6
-rwxr-xr-x. 1 root root      228 5月  27 11:09 pip3
-rwxr-xr-x. 1 root root      228 5月  27 11:09 pip3.6
lrwxrwxrwx. 1 root root        8 5月  27 11:09 pydoc3 -> pydoc3.6
-rwxr-xr-x. 1 root root       88 5月  27 11:09 pydoc3.6
lrwxrwxrwx. 1 root root        9 5月  27 11:09 python3 -> python3.6
-rwxr-xr-x. 2 root root 12711488 5月  27 11:09 python3.6
lrwxrwxrwx. 1 root root       17 5月  27 11:09 python3.6-config -> python3.6m-config
-rwxr-xr-x. 2 root root 12711488 5月  27 11:09 python3.6m
-rwxr-xr-x. 1 root root     3101 5月  27 11:09 python3.6m-config
lrwxrwxrwx. 1 root root       16 5月  27 11:09 python3-config -> python3.6-config
lrwxrwxrwx. 1 root root       10 5月  27 11:09 pyvenv -> pyvenv-3.6
-rwxr-xr-x. 1 root root      445 5月  27 11:09 pyvenv-3.6
[root@master-61 opt]# ls -l /opt/venv_py3/bin/
总用量 32
-rw-r--r--. 1 root root 2197 5月  27 11:31 activate
-rw-r--r--. 1 root root 1253 5月  27 11:31 activate.csh
-rw-r--r--. 1 root root 2417 5月  27 11:31 activate.fish
-rwxr-xr-x. 1 root root  243 5月  27 11:31 easy_install
-rwxr-xr-x. 1 root root  243 5月  27 11:31 easy_install-3.6
-rwxr-xr-x. 1 root root  225 5月  27 11:31 pip
-rwxr-xr-x. 1 root root  225 5月  27 11:31 pip3
-rwxr-xr-x. 1 root root  225 5月  27 11:31 pip3.6
lrwxrwxrwx. 1 root root    7 5月  27 11:31 python -> python3
lrwxrwxrwx. 1 root root   26 5月  27 11:31 python3 -> /opt/python369/bin/python3
[root@master-61 opt]# 
```



#### 虚拟环境的作用

```
一个python项目，需要装很多依赖
另一个也需要装很多的依赖，如果公用同一个python本体解释器，会导致依赖冲突，版本互相之间冲突。

#使用pip3安装所需要的模块，都会安装到这个目录下面
[root@master-61 ~]# cd /opt/python369/lib/python3.6/site-packages/




# 1.激活虚拟环境
 source /opt/venv_py3/bin/activate

# 2.找到项目依赖文件
(venv_py3) [root@master-61 requirements]# ls -l /opt/jumpserver-v2.12.0/requirements/requirements.txt 
-rw-rw-r--. 1 root root 2199 Jul 15  2021 /opt/jumpserver-v2.12.0/requirements/requirements.txt


# 3.查看当前环境下的依赖
(venv_py3) [root@master-61 requirements]# pip list
Package    Version
---------- -------
pip        18.1   
setuptools 40.6.2 
You are using pip version 18.1, however version 21.3.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
(venv_py3) [root@master-61 requirements]# 

# 4.安装依赖
(venv_py3) [root@master-61 requirements]# pip3 install -r /opt/jumpserver-v2.12.0/requirements/requirements.txt 



```





## 设置pip3的下载源，豆瓣源，加速模块下载

```
创建文件夹
mkdir ~/.pip


cat > ~/.pip/pip.conf <<EOF
[global]
index-url=https://pypi.douban.com/simple
EOF
```





## python环境安装好，可以运行项目了

### 修改jumpserver代码的配置文件

```
1.部署core服务，后端python服务的时候，需要全程激活虚拟环境。
激活虚拟环境，安装jumpserver后端所需的python依赖
[root@master-61 ~]# source /opt/venv_py3/bin/activate
(venv_py3) [root@master-61 ~]# 


# 2.拷贝配置文件
(venv_py3) [root@master-61 ~]# cd /opt/jumpserver-v2.12.0/

(venv_py3) [root@master-61 jumpserver-v2.12.0]# ls
apps                data        docs           jms      logs          README.md     run_server.py  utils
config_example.yml  Dockerfile  entrypoint.sh  LICENSE  README_EN.md  requirements  tmp            Vagrantfile
(venv_py3) [root@master-61 jumpserver-v2.12.0]# cp config_example.yml config.yml 
(venv_py3) [root@master-61 jumpserver-v2.12.0]# ls
apps                config.yml  Dockerfile  entrypoint.sh  LICENSE  README_EN.md  requirements   tmp    Vagrantfile
config_example.yml  data        docs        jms            logs     README.md     run_server.py  utils
(venv_py3) [root@master-61 jumpserver-v2.12.0]# 


# 3.修改配置文件如下
vim /opt/jumpserver-v2.12.0/config.yml
jumpserver整个架构的所有组件，相互之间的通信，都是基于一个秘钥来加密的。
基于如下命令，生成2个秘钥


```

