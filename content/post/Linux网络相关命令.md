---
title: "网络相关"
slug: "网络相关"
date: "2024-04-13"
created: "2024-04-13"
tags: ["OP"]
---


Linux网络命令



```
ip
ip addr
ip link 
ip route
ifconfig
dig
nslookup
netstat
traceroute
tracepath
host
hostname
ping
ll  ss
route
arp
iwconfig
curl
wget
mtr
whois
iftop
tcpdump
ifplugstatus
```



## ip

```
语法：
ip   [options]   object   [command]
[options] - 定义修改命令行为;
object - 表示可用于配置的对象;
[command] - 子命令，是对对象执行的操作，可用的命令因对象而异;

示例：
[root@node ~]# ip
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
where  OBJECT := { address | addrlabel | fou | help | ila | l2tp | li
                   macsec | maddress | monitor | mptcp | mroute | mru
                   neighbor | neighbour | netconf | netns | nexthop |
                   ntbl | route | rule | sr | tap | tcpmetrics |
                   token | tunnel | tuntap | vrf | xfrm }
       OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[esol
                    -h[uman-readable] | -iec | -j[son] | -p[retty] |
                    -f[amily] { inet | inet6 | mpls | bridge | link }
                    -4 | -6 | -I | -D | -M | -B | -0 |
                    -l[oops] { maximum-addr-flush-attempts } | -br[ie
                    -o[neline] | -t[imestamp] | -ts[hort] | -b[atch] 
                    -rc[vbuf] [size] | -n[etns] name | -N[umeric] | -
                    -c[olor]}
[root@node ~]# 
```

## ip   addr

```
语法：
ip  addr   [subcommand]
[subcommand] 操作:
add - 添加新地址;
show - 显示协议地址;
del - 删除地址;
flush - 根据指定标准刷新地址;

[root@node ~]# ip addr show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 52:54:00:2d:74:35 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    altname ens5
    inet 10.0.4.12/22 brd 10.0.7.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe2d:7435/64 scope link 
       valid_lft forever preferred_lft forever
```

## ip route

`ip route` 命令显示并配置 IP 路由表。通过该命令，用户可以调整路由表，并利用路由表执行其他网络任务。

```
ip  route  [subcommand]  [options]  [destination] 
命令组成包括：
[subcommand] 操作:
show - 显示路由表；
add - 向表中添加一条新路由；
del - 从表中删除路由；
change - 修改现有路由；
[destination]： - 决定网络流量的方向

[root@node ~]# ip route show
default via 10.0.4.1 dev eth0 proto dhcp metric 100 
10.0.4.0/22 dev eth0 proto kernel scope link src 10.0.4.12 metric 100 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
```

## dig

（命令用于查询域名系统（DNS）和查找 DNS 记录信息。）

```
#Debian(Ubuntu,kili)
apt-get  install dnsutils
#redhat(CentOS/Fedora)
yum -y install bind-utils
语法：

dig [options] [domain] [record type] [DNS server]
命令组成包括：

[options] - 用于修改命令行为的参数；

[domain] - 要查询的域名；

[record type] - 要查询的 DNS 记录类型，默认为 A 记录；

[DNS server] - 用于查询的指定 DNS 服务器；

[root@node ~]# dig bing.com

; <<>> DiG 9.16.23-RH <<>> bing.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16572
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;bing.com.			IN	A

;; ANSWER SECTION:
bing.com.		1025	IN	A	13.107.21.200
bing.com.		1025	IN	A	204.79.197.200

;; Query time: 1 msec
;; SERVER: 183.60.83.19#53(183.60.83.19)
;; WHEN: Tue Jan 02 23:13:43 CST 2024
;; MSG SIZE  rcvd: 58

#提供 IP 地址和 -x 选项以执行反向 DNS 查找。
[root@node ~]# dig -x 8.8.8.8
```

## nslookup

`nslookup` 命令与 `dig` 命令类似。这两条命令的主要区别在于 `nslookup` 具有交互模式。它可以诊断和查询 DNS 服务器，有助于网络故障排除和执行相关 DNS 任务。

```
[root@node ~]# nslookup  baidu.com
Server:		183.60.83.19
Address:	183.60.83.19#53

Non-authoritative answer:
Name:	baidu.com
Address: 110.242.68.66
Name:	baidu.com
Address: 39.156.66.10
```

## netstat

`netstat` 命令（network statistics）是一个网络实用程序，用于显示各种网络统计信息,该命令提供网络端口的统计数据，并显示端口的可用性。

```
net-tools 软件包的一部分 
可以使用iproute2 中的 ss 命令来替代
#显示系统中所有活动的 TCP 连接。
[root@node ~]# netstat  -at
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:sunrpc          0.0.0.0:*               LISTEN     
tcp        0      0 node:45430              169.254.0.55:lsi-bobcat ESTABLISHED
tcp        0      0 node:ssh                ip1f134b5c.dynami:53586 TIME_WAIT  
tcp        0      0 node:45426              169.254.0.55:lsi-bobcat ESTABLISHED
tcp        0      0 node:42614              169.254.0.138:8186      ESTABLISHED
tcp        0      0 node:55894              169.254.0.4:http        TIME_WAIT  
tcp        0    232 node:ssh                58.19.1.45:25192        ESTABLISHED
tcp        0      0 node:ssh                129.226.210.126:46448   ESTABLISHED
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN     
tcp6       0      0 [::]:sunrpc             [::]:*                  LISTEN   
```

## traceroute和tracepath

`traceroute` 命令是一种网络诊断工具，适用于 Linux、macOS 和 Windows。该命令可跟踪数据包到达 TCP/IP 网络目的地的路径。该命令通过显示数据包从源到目标传输时的中间跃点来发现路由问题和瓶颈。默认跟踪为 30 跳，IPv4 数据包大小为 60 字节（IPv6 为 80 字节）。

```
语法:

traceroute [options] [hostname/IP]
命令组成包括：

[hostname/IP]- 域名/IP，必需，其他选项则控制是否执行 DNS 查找、TTL 参数和数据包类型;

示例：

要使用 TCP 协议跟踪数据包路由，请以管理员身份运行 traceroute 命令，并使用 -T 选项。例如:

wget http://mirror.centos.org/centos/7/os/x86_64/Packages/traceroute-2.0.22-2.el7.x86_64.rpm


[root@node ~]# sudo traceroute -T 184.95.56.34
```

## host

`host` 命令是执行 DNS 查询的简单工具。该命令可将 IP 地址解析为域名，反之亦然。使用该命令执行 DNS 记录查询和基本 DNS 故障排除。

## ping

`ping` 命令是一种网络工具，用于测试主机是否可连接，该命令向主机（计算机或服务器）发送 ICMP 请求，并测量往返时间（RTT）。 

`ping` 有助于确定两个节点之间的网络延迟，以及网络是否可以到达。

## ss

`ss` 命令是用于显示网络统计信息的 CLI 工具，该工具是 `iproute2` 软件包的一部分，是 `netstat` 命令的更快替代方法。

使用 ss 命令检查网络套接字并查看各种网络相关数据。

```
[root@node ~]# ss -lt
State             Recv-Q            Send-Q                       Local Address:Port                          Peer Address:Port            Process            
LISTEN            0                 128                                0.0.0.0:ssh                                0.0.0.0:*                                  
LISTEN            0                 4096                               0.0.0.0:sunrpc                             0.0.0.0:*                                  
LISTEN            0                 128                                   [::]:ssh                                   [::]:*                                  
LISTEN            0                 4096                                  [::]:sunrpc                                [::]:*                                  
[root@node ~]# 

```

Linux 中的 `route` 命令是用于显示和配置路由表的专用命令。该命令修改内核的 IP 路由表，并帮助设置到特定主机或网络的静态路由。

**注意：**`route` 命令的最佳替代命令是 `ip route` 命令。

```
[root@node ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    100    0        0 eth0
10.0.4.0        0.0.0.0         255.255.252.0   U     100    0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
[root@node ~]# ip route
default via 10.0.4.1 dev eth0 proto dhcp metric 100 
10.0.4.0/22 dev eth0 proto kernel scope link src 10.0.4.12 metric 100 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
[root@node ~]# 
#添加默认网关
sudo route add default gw [gateway]

```

## arp

`arp` 命令 是 `Address Resolution Protocol`，地址解析协议，是通过解析网络层地址来找寻数据链路层地址的一个网络协议包中极其重要的网络传输协议。而该命令可以显示和修改 arp 协议解析表中的缓冲数据。

## iwconfig

`iwconfig` 命令用于显示和配置无线网络接口信息，该命令在排除无线网络问题时非常有用。通过使用该命令可以查看或更改无线网络名称、电源管理设置和其他无线配置。

##  curl 或 wget

`wget` 和 `curl` 是从互联网下载文件的命令行工具。这两个工具很相似，但工作方式和提供的选项略有不同：

- `wget` 命令使用 HTTP、HTTPS 或 FTP 协议从网上下载文件;
- `curl` 命令用途广泛，支持各种网络协议，如 SCP、IMAP POP3、SMTP 等。该工具还能发送 HTTP 请求并与网络服务交互;

使用 `curl` 或 `wget` 测试网络下载速度。

```
wget -O [file name] [URL]
```

```
curl -o [file name] [URL]
```

## mtr

`mtr` 命令（my traceroute）是一种诊断工具，结合了 `ping` 和 `traceroute` 命令的元素。该命令可实时显示网络质量，是排除高延迟和丢包故障的绝佳工具。

```
[root@node ~]# yum -y install  mtr
[root@node ~]# mtr bing.com

```


## whois

`whois` 命令用于查询域名、IP 地址和其他网络相关信息。使用该命令可获取域名所有权详情，如域名所有权详情、注册日期和到期日期。

```
[root@node ~]# yum install -y  whois

```

## iftop(网络流量监控利器)

iftop 命令是一种网络监控工具。使用该命令可实时查看网络连接和带宽使用情况。

```
# 安装比较软件包
$ sudo um install libpcap libpcap-devel ncurses ncurses-devel flex byacc

# 下载软件包
wget "http://www.ex-parrot.com/~pdw/iftop/download/iftop-0.17.tar.gz"
$ tar zxvf iftop-0.17.tar.gz
$ cd iftop-0.17
$ ./configure
$ make &amp;&amp; make install

```

