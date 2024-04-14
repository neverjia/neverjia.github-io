---
title: test
slug: test
tags: [ K8s, Docker, Grafana ]
date: 2023-03-17T20:50:20+08:00
---

```
cat  >>/etc/hosts <<'EOF'
192.168.230.200 kmaster
192.168.230.201 knode1
192.168.230.202 knode2
EOF

```
### 防火墙初始化

```
systemctl stop firewalld NetworkManager
systemctl disable firewalld NetworkManager

sed -ri 's#(SELINUX=).*#\1disabled#' /etc/selinux/config
setenforce 0
systemctl disable firewalld && systemctl stop firewalld


getenforce 0

#清空所有防火墙规则
iptables -F
#删除所有自定义链，即删除所有用户自定义的链
iptables -X
#这个命令用于重置所有计数器，即将数据包和字节计数器归零
iptables -Z
# 这个命令用于设置默认的转发策略为接受（ACCEPT），即允许所有转发的数据包通过防火墙。-P FORWARD ACCEPT 将转发链的默认策略设置为接受。
iptables -P FORWARD ACCEPT
```

### 关闭Swap
k8s默认禁用swap功能
```
swapoff -a
# 防止开机自动挂载 swap 分区
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```