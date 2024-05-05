---
title: "Prometheus安装与使用"
slug: "Prometheus安装与使用"
date: "2024-05-04"
created: "2024-05-04"
tags: ["Prometheus"]
---


> Prometheus作为数据源，采集中心,用于收集各node节点的监控数据。
>
> 使用客户端使用node_exporter(其他的exporter)向Prometheus，或直接通过Pushgateway推送监控数据。Grafana用于图表展示监控数据,Alertmanager用于告警。



# 1.工作流程

![image](https://img-blog.csdnimg.cn/34630c9fd299495a88f1b7a3ca10c6c5.png)

# 2.镜像下载

> https://github.com/prometheus 

## 1.下载并安装二进制版本prometheus

```
#最新2.51.2 / 2024-04-09 Latest
#下载链接https://github.com/prometheus/prometheus/releases/download/v2.52.0-rc.1/prometheus-2.52.0-rc.1.linux-amd64.tar.gz
VERSION=2.42.0
wget https://github.com/prometheus/prometheus/releases/download/V${VERSION}/prometheus-${VERSION}.linux-amd64.tar.gz
tar xzvf prometheus-${VERSION}.1inux-amd64.tar.gz
mv prometheus-${VERSIoN}.linux-amd64  /usr/local/prometheus

```

### 1.1配置Systemd服务，实现开机自启动(Centos)

```
#解压安装
[root@localhost ~]# tar -zxvf prometheus-2.52.0-rc.1.linux-amd64.tar.gz

[root@localhost ~]# mv prometheus-2.52.0-rc.1.linux-amd64 /usr/local/prometheus
[root@localhost ~]# cd /usr/local/prometheus/
[root@localhost prometheus]# ll
total 262500
drwxr-xr-x. 2 1001 127        38 May  3 14:00 console_libraries
drwxr-xr-x. 2 1001 127       173 May  3 14:00 consoles
-rw-r--r--. 1 1001 127     11357 May  3 14:00 LICENSE
-rw-r--r--. 1 1001 127      3773 May  3 14:00 NOTICE
-rwxr-xr-x. 1 1001 127 138433773 May  3 13:46 prometheus
-rw-r--r--. 1 1001 127       934 May  3 14:00 prometheus.yml
-rwxr-xr-x. 1 1001 127 130340476 May  3 13:46 promtool
[root@localhost prometheus]# pwd
/usr/local/prometheus
[root@localhost prometheus]# chown -R root:root /usr/local/prometheus/
[root@localhost prometheus]# cat>/usr/lib/systemd/system/prometheus.service<<EOF
> [Unit]
> Description=Prometheus
> After=network.target
> 
> [Service]
> Type=simple
> User=root
> WorkingDirectory=/usr/local/prometheus
> ExecStart=/usr/local/prometheus/prometheus --web.enable-lifecycle --
> config.file=/usr/local/prometheus/prometheus.yl --storage.tsdb.retention=7d
> 
> Restart=on-failure
> LimitNOFILE=65536
> 
> [Install]
> WantedBy=multi-user.target
> EOF
[root@localhost prometheus]# systemctl daemon-reload
[root@localhost prometheus]# systemctl enable prometheus.service && systemctl start prometheus.service
Created symlink from /etc/systemd/system/multi-user.target.wants/prometheus.service to /usr/lib/systemd/system/prometheus.service.
[root@localhost prometheus]# 

```

### 1.2查看端口

```
[root@localhost ~]# ss -tunlp | grep prometheus
tcp    LISTEN     0      128    [::]:9090               [::]:*                   users:(("prometheus",pid=9648,fd=7))

#查看日志
[root@localhost ~]# tail -f /var/log/messages

#prometheus的配置文件是prometheus.yml
#用空格缩进，2个，备注换行备注
[root@localhost ~]# cd /usr/local/prometheus/
[root@localhost prometheus]# ll
total 262500
drwxr-xr-x. 2 root root        38 May  3 14:00 console_libraries
drwxr-xr-x. 2 root root       173 May  3 14:00 consoles
drwxr-xr-x. 4 root root        70 May  5 02:06 data
-rw-r--r--. 1 root root     11357 May  3 14:00 LICENSE
-rw-r--r--. 1 root root      3773 May  3 14:00 NOTICE
-rwxr-xr-x. 1 root root 138433773 May  3 13:46 prometheus
-rw-r--r--. 1 root root       934 May  3 14:00 prometheus.yml
-rwxr-xr-x. 1 root root 130340476 May  3 13:46 promtool
[root@localhost prometheus]# 
```

### 1.3配置文件位置prometheus.yml

```
[root@node1 prometheus]# cat /usr/local/prometheus/prometheus.yml
```



## 2.节点node_exporter配置(客户端配置)


### 2.1下载并安装二进制版本
```
#下载链接https://github.com/prometheus/node_exporter/releases/download/v1.8.0/node_exporter-1.8.0.linux-amd64.tar.gz
VERSION=1.5.0
wget
https ://github.com/prometheus/node_exporter/releases/download/V${VERSION}/node_exporter-${VERSION}.linux-amd64.tar.gz
tar xzvf node_exporter-${VERSION}.linux-amd64.tar.gzmv node_exporter-${VERSION}.1inux-amd64 /usr/local/node_exporter/
```
### 2.2解压安装

```
#自启配置文件 
cat>/usr/lib/systemd/system/node_exporter.service<<EOF
[Unit]
Description=Node Exporter
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/node_exporter/node_exporter

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

#开机自启
systemctl daemon-reload
 systemctl enable node_exporter.service && systemctl start node_exporter.service
```

### 2.3查看端口

```
[root@node2 ~]# ss -tunlp | grep 9100
tcp    LISTEN     0      128    [::]:9100               [::]:*                   users:(("node_exporter",pid=9597,fd=3))
[root@node2 ~]# 
```

### 2.4服务端配置

```
job_name: 'node2'     #增加一个节点，取名为 node2自己随便取，方便记忆即可
etrics_path:"/metricstatic_configs :      # 获取数据的路径 http://10.0.20.12:9100/metrics
targets:['10.0.20.12:9100'] # 静态配置 node 节点的地址

#配置文件编辑
[root@node1 prometheus]# cat prometheus.yml
# my global config
global:
  scrape_interval: 15s 
  evaluation_interval: 15s 
alerting:
  alertmanagers:
    - static_configs:
        - targets:
rule_files:
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["192.168.230.176:9090"]
##采集客户端添加如下
  - job_name: 'node2'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['192.168.230.177:9100']
[root@node1 prometheus]# 


```

### 2.5配置文件

```
 cd /usr/local/node_exporter/

```



## 3.管理配置prometheus信息

### 3.1prometheus管理API

> prometheus提供了一套管理API来简化自动化和集成。

```
#健康检查
#这个端点总是返回200，应该用来检查prometheus的监控状况。
curl -X GET http://192.168.230.176:9090/-/healthy

#就绪检查
#当prometheus准备好为流量服务（即响应查询）时，此端点返回200
curl -X GET http://192.168.230.176:9090/-/

#重新加载
此端点触发重新加载prometheus配置和规则文件夹。默认情况下是禁用的，可以通过--web.enable-lifecycle选项来启用 它
curl -X PUT http://192.168.230.176:9090/-/reload
curl -X POST http://192.168.230.176:9090/-/reload


[root@localhost prometheus]# curl -X GET http://192.168.230.176:9090/-/healthy
Prometheus Server is Healthy.
[root@localhost prometheus]# curl -X GET http://192.168.230.176:9090/-/ready
Prometheus Server is Ready.
[root@localhost prometheus]# 
```

### 3.2检查配置文件

```
#/usr/local/prometheus/promtool check config /usr/local/prometheus/prometheus.yml
[root@localhost prometheus]# /usr/local/prometheus/promtool check config /usr/local/prometheus/prometheus.yml
Checking /usr/local/prometheus/prometheus.yml
 SUCCESS: /usr/local/prometheus/prometheus.yml is valid prometheus config file syntax

#这样重启很慢
#systemctl restart prometheus.service
#可进行热重启
curl -X PUT http://192.168.230.176:9090/-/reload
```

### 3.3prometheus自动发现-给予文件 发现配置添加各节点node信息

```
创建对应的目录
[root@node1 prometheus]# pwd
/usr/local/prometheus

# 创建targets目录，并且创建对应的子目录，用于存放相应的配置文件自动发现
[root@node1 prometheus]# mkdir -p targets/{docker,nodes}
```

```
#在创建好的nodes目录下创建 nodes.json文件，并写入下面内容[root@es01 config]# cat targets/nodes/nodes.json
[root@node1 nodes]# pwd
/usr/local/prometheus/targets/nodes
[root@node1 prometheus]# cat targets/nodes/node.json 
[{
  "targets": [
     "192.168.230.176:9100",
     "192.168.230.177:9100",
     "192.168.230.178:9100"
    ],
    "labels": {
       "server": "node_export01"
    }
  }]
[root@node1 prometheus]# 

[root@node1 nodes]# pwd
/usr/local/prometheus/targets/nodes
[root@node1 nodes]# 

```



### 3.4prometheus配置文件就需要重新编写

```
#原来1台客户端的配置
[root@node1 prometheus]# cat prometheus.yml
# my global config
global:
  scrape_interval: 15s 
  evaluation_interval: 15s 
alerting:
  alertmanagers:
    - static_configs:
        - targets:
rule_files:
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["192.168.230.176:9090"]
  - job_name: 'node2'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['192.168.230.177:9100']

#现在集群配置
[root@node1 prometheus]# cat prometheus.yml
# my global config
global:
  scrape_interval: 15s 
  evaluation_interval: 15s 
alerting:
  alertmanagers:
    - static_configs:
        - targets:
rule_files:
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["192.168.230.176:9090"]

#  - job_name: 'node2'
#    metrics_path: '/metrics'
#    static_configs:
#      - targets: ['192.168.230.177:9100']
  - job_name: "node"
    metrics_path: "/metrics"
    file_sd_configs:
      - files:
          - targets/nodes/*.json
        refresh_interval: 1m
[root@node1 prometheus]# 
      
      
#解释含义
rule_files:
scrape_configs:                         #这个配置表示静态发现
  - job_name: "prometheus"
    static_configs:
      - targets: ["192.168.230.176:9090"]
  - job_name: 'node2'                   #增加一个节点，取名为node2
    metrics_path: '/metrics'            #获取数据的路径http://192.168.230.177:9100/metrics
    file_sd_configs:                    #这个配置表示通过文件发现
        - files:       
          - targets/nodes/*.json            #读取targets/nodes目录下面的所有json结尾的文件
      refresh_interval: 1m              #刷新频率1分钟
```



### 3.5检查，重新加载配置文件

```
#/usr/local/prometheus/promtool check config /usr/local/prometheus/prometheus.yml
[root@localhost prometheus]# /usr/local/prometheus/promtool check config /usr/local/prometheus/prometheus.yml
Checking /usr/local/prometheus/prometheus.yml
 SUCCESS: /usr/local/prometheus/prometheus.yml is valid prometheus config file syntax

#这样重启很慢
#systemctl restart prometheus.service
#可进行热重启
curl -X PUT http://192.168.230.176:9090/-/reload
```

## 4.grafana配置

```
wget https://dl.grafana.com/enterprise/release/grafana-enterprise-10.4.2.linux-amd64.tar.gz
tar -zxvf grafana-enterprise-10.4.2.linux-amd64.tar.gz
```

### 4.1创建systemd服务

```
#自启配置文件 
cat>/usr/lib/systemd/system/grafana-server.service<<EOF
[Unit]
Description=Grafana Server
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=root
WorkingDirectory=/usr/local/grafana
ExecStart=/usr/local/grafana/bin/grafana-server

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

#开机自启
systemctl daemon-reload
 systemctl enable grafana-server.service && systemctl start grafana-server.service
 
```

### 4.2查看端口

```
[root@node1 grafana]# ss -tunlp | grep grafana
tcp    LISTEN     0      128    [::]:3000               [::]:*                   users:(("grafana",pid=10338,fd=11))

```

### 4.3查看日志

```
[root@node1 grafana]# tail -f /var/log/messages 
May  5 04:23:28 localhost grafana-server: logger=grafanaStorageLogger t=2024-05-05T04:23:28.099472566-04:00 level=info msg="Storage starting"
May  5 04:23:28 localhost grafana-server: logger=ngalert.multiorg.alertmanager t=2024-05-05T04:23:28.121065345-04:00 level=info msg="Starting MultiOrg Alertmanager"
May  5 04:23:28 localhost grafana-server: logger=ngalert.scheduler t=2024-05-05T04:23:28.121148622-04:00 level=info msg="Starting scheduler" tickInterval=10s maxAttempts=1
May  5 04:23:28 localhost grafana-server: logger=ticker t=2024-05-05T04:23:28.121212653-04:00 level=info msg=starting first_tick=2024-05-05T04:23:30-04:00
```

### 4.4配置文件defaults.ini

```
[root@node1 conf]# pwd
/usr/local/grafana/conf
[root@node1 conf]# ll
total 180
-rw-r--r--. 1 root root 88248 Apr 10 11:10 defaults.ini
-rw-r--r--. 1 root root  1045 Apr 10 11:09 ldap_multiple.toml
-rw-r--r--. 1 root root  2986 Apr 10 11:09 ldap.toml
drwxr-xr-x. 8 root root   113 Apr 10 11:09 provisioning
-rw-r--r--. 1 root root 83548 Apr 10 11:10 sample.ini
[root@nod
```

### 4.5监控查看效果

```
#网页登录http://192.168.230.176:3000/
用户名密码默认
admin
admin
```

### 4.6添加prometheus数据源

> 设置-Date sources

### 4.7添加面板-生成可视化图表


## 5.安装node_exporter的脚本

```
#!/bin/bash
#1 firewall & selinux这是测试环境可以关闭，生成环境不建议。
echo '###01 Checking Firewall and SELinux###'
systemctl stop firewalld
systemctl disable firewalld
se01="SELINUX=disabled"
se02=$(cat /etc/selinux/config |grep -w "^SELINUX")
if [[ "$se01" == "$se02" ]]
then
	echo "Configuration passed "
else
	echo "SELinux Not Closed,configuring......"
	sed -i 's/^SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
	echo "configuration successful "
fi
echo "03 configuration successful "

#2.下载基础软件
yum install -y yum-utils vim bash-completion net-tools wget

#下载node_exporter安装包
#wget  https://github.com/prometheus/node_exporter/releases/download/v1.8.0/node_exporter-1.8.0.linux-amd64.tar.gz

VERSION=1.8.0
wget   https://github.com/prometheus/node_exporter/releases/download/V${VERSION}/node_exporter-${VERSION}.linux-amd64.tar.gz
tar -xzvf node_exporter-${VERSION}.linux-amd64.tar.gzmv node_exporter-${VERSION}.1inux-amd64 /usr/local/node_exporter/
mv node_exporter-1.8.0.linux-amd64 /usr/local/node_exporter/
chown -R root:root /usr/local/node_exporter/

#自启配置文件 
cat>/usr/lib/systemd/system/node_exporter.service<<EOF
[Unit]
Description=Node Exporter
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/node_exporter/node_exporter

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

#开机自启
systemctl daemon-reload
systemctl enable node_exporter.service && systemctl start node_exporter.service

#查看监听状态
ss -tunlp | grep 9100
```







 
