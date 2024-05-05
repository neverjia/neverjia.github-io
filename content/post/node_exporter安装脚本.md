
---
title: "node_exporter安装脚本"
slug: "node_exporter安装脚本"
date: "2024-05-05"
created: "2024-05-05"
tags: ["node_exporter"]
---

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