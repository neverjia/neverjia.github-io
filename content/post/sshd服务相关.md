
---
title: "sshd服务相关"
slug: "sshd服务相关"
date: "2024-04-13"
created: "2024-04-13"
tags: ["sshd"]
---




# 关于OpenSSH服务（sshd）相关的

# 一、Ubuntu操作系统

## 1.安装OpenSSH服务器（Ubuntu）

```
sudo  apt update
sudo  apt install openssh-server
```

## 2.启动和停止服务（Ubuntu）

```
sudo service ssh start        # 启动 SSH 服务
sudo service ssh stop         # 停止 SSH 服务
sudo service ssh restart      # 重启 SSH 服务
sudo service ssh status       # 检查 SSH 服务状态
```

## 3.修改ssh服务配置文件（Ubuntu）

```
sudo nano /etc/ssh/sshd_config    # 使用 nano 编辑器打开配置文件
或者vim /etc/ssh/sshd_config
# 在文件中进行相应更改，例如：
Port 22                           # 修改 SSH 服务监听的端口号
PermitRootLogin yes               # 允许root 用户远程登录
PasswordAuthentication yes        # 启用密码身份验证

#usePAM 表示是否启用 Pluggable Authentication Modules（PAM）作为身份验证机制的一部分。PAM允许系统管理员配置和控制不同类型的认证策略，比如密码复杂度验证、账号锁定、多因素认证等
usePAM  no

#重启sshd服务
systemctl restart sshd




# 保存并关闭文件后，重启 SSH 服务以使更改生效
sudo service ssh restart
```

## 4.查看SSH服务状态及错误日志

```
sudo service sshd status
或者systemctl  status sshd

#实时监视SSH认证
sudo  tail -f  /var/log/auth.log
#实时监听系统日志
sudo  tail -f  /var/log/syslog


```

## 5.使用UFW防火墙允许ssh连接

```
sudo  ufw allow  ssh
```

## 6.SSH客户端连接

```
ssh  127.0.0.1
ssh  username@server.ip
```





# 二、在 CentOS 操作系统中安装 SSH 服务（OpenSSH）的步骤如下：

## 1.更新安装包

```
sudo  yum update
```

## 2.安装服务

```
sudo  yum -y install  openssh-server
```

## 3.启动服务

```
sudo  systemctl status sshd
```

## 4.设置SSH开机自启

```
sudo systemctl enable  sshd
```

> 安装完成后，您可以根据需要修改 `/etc/ssh/sshd_config` 文件来调整 SSH 服务器的配置。

## 允许防火墙

```
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```









### 修改了/etc/hosts.deny和/etc/hosts.allow怎么排错

```

当远程连接无法建立并且用户修改了 `/etc/hosts.deny` 和 `/etc/hosts.allow` 配置文件后，可以按照以下步骤进行排查和调试：

1. 使用本地访问：首先，尝试在本地主机上使用 SSH 客户端连接到该服务器。这将帮助确定是远程连接问题还是网络问题。
2. 检查配置文件更改：确保 `/etc/hosts.deny` 和 `/etc/hosts.allow` 文件的更改是正确的，并且没有错误的语法或配置问题。验证文件的权限和所有者是否正确。
3. 检查访问规则：检查 `/etc/hosts.deny` 和 `/etc/hosts.allow` 文件中的访问规则是否正确。确保没有设置过于严格的规则阻止了您的远程连接。可以使用文本编辑器打开这些文件进行检查。
4. 查看系统日志：查看系统日志文件以获取关于连接问题的更多信息。系统日志文件通常位于 `/var/log/syslog`、`/var/log/auth.log` 或 `/var/log/messages` 等位置。使用命令 `sudo tail -f /var/log/syslog` 或 `sudo tail -f /var/log/auth.log` 来实时监视日志文件并查看错误消息。
5. 检查防火墙设置：如果系统中启用了防火墙（如 iptables 或 UFW），请检查防火墙配置以确保远程连接的流量被允许通过。确保正确配置了适当的端口和协议。
6. 检查网络连接：确保网络连接正常，没有任何问题。尝试使用 ping 命令测试服务器的可达性，并检查网络配置是否正确。
7. 检查 SSH 服务状态：使用命令 `sudo service ssh status` 或 `sudo systemctl status ssh` 来检查 SSH 服务的状态。确保服务正在运行，并检查是否有任何错误或警告信息。
8. 检查 SSH 配置：查看 SSH 服务的配置文件 `/etc/ssh/sshd_config`，确保配置正确，并且没有禁用远程连接或使用不正确的参数。
9. 尝试使用其他 SSH 客户端：如果您正在使用特定的 SSH 客户端进行连接尝试，请尝试使用其他 SSH 客户端进行连接，例如 OpenSSH 客户端或 PuTTY。这将帮助确定是否是客户端软件的问题。

```





### /etc/hosts.deny和/etc/hosts.allow配置文件作用
```

- `/etc/hosts.deny`：这个文件用于指定不允许访问主机的规则。您可以在这里定义禁止访问的 IP 地址、主机名或网络。例如，如果要禁止来自特定 IP 地址的访问，可以在 `/etc/hosts.deny` 文件中添加以下行：

```

```
sshd: 192.168.1.100
这将阻止 IP 地址为 192.168.1.100 的主机访问 SSH 服务。

- `/etc/hosts.allow`：与 `/etc/hosts.deny` 相反，这个文件用于指定允许访问主机的规则。您可以在这里定义允许访问的 IP 地址、主机名或网络。例如，如果要允许来自特定 IP 地址的访问，可以在 `/etc/hosts.allow` 文件中添加以下行：


```

```
sshd: 192.168.1.200
这将允许 IP 地址为 192.168.1.200 的主机访问 SSH 服务。

请注意，如果同时在 `/etc/hosts.deny` 和 `/etc/hosts.allow` 中定义了相同的规则，`/etc/hosts.allow` 中的规则将优先生效。

如果您在系统中找不到这两个文件，您可以手动创建它们。请确保在编辑这些文件之前备份它们，以防止意外的配置错误。






```
   

### debug

> 1.查看系统日志：查看系统日志文件以获取关于连接问题的更多信息。系统日志文件通常位于 `/var/log/syslog`、`/var/log/auth.log` 或 `/var/log/messages` 等位置。使用命令 `sudo tail -f /var/log/syslog` 或 `sudo tail -f /var/log/auth.log` 来实时监视日志文件并查看错误消息。

```
#相关命令 

ssh  -vvvv 127.0.0.1
   ss  -ntlp  |  grep  22
   systemctl stop  ssd
   ps -ef  |  grep sshd
   ss  -tnlp |  grep 
   iptable -vnL
```

#### ssh  -vv的解释

   ```
ssh  -vvvv 127.0.0.1
通常情况下，使用 -v（一个或两个 -v）就足够了
ssh：这是 SSH 客户端的命令，用于建立 SSH 连接。
-vvvv：这是调试选项，用于增加详细的调试输出。每个 -v 都会增加一层详细程度，因此 -vvvv 表示使用非常详细的调试模式。通过使用多个 -v，您可以在调试输出中获得更多的细节，以便更好地了解 SSH 连接的过程和问题。
127.0.0.1：这是目标主机的 IP 地址或主机名。在这个例子中，使用的是本地主机的回环地址，即 127.0.0.1。这个地址用于访问本机上的 SSH 服务。



   ```
## ss  -ntlp  |  grep  22

```
ss -ntlp | grep 22 是一个命令示例，用于列出当前正在监听端口 22 的网络连接。

解释该命令的各个部分如下：

ss：这是一个强大的命令行工具，用于查看套接字统计信息，包括网络连接、套接字状态等。
-ntlp：这是 ss 命令的选项，用于指定显示的信息类型和过滤条件。
-n：禁用域名解析，以使用 IP 地址而不是主机名。
-t：仅显示 TCP 连接。
-l：仅显示监听（listening）的套接字。
-p：显示进程标识符（PID）和进程名称，以便识别每个连接所属的进程。
|：管道符号，用于将前一个命令的输出作为后一个命令的输入。
grep 22：这是一个用于文本搜索的命令，它筛选并显示包含数字 22 的行。在这种情况下，它用于筛选包含端口号 22 的连接信息。
```
## ps -ef  |  grep sshd

```
ps -ef | grep sshd 是一个命令示例，用于查找正在运行的与 SSH 服务相关的进程。

解释该命令的各个部分如下：

ps：这是一个用于显示当前正在运行的进程的命令。
-ef：这是 ps 命令的选项，用于指定以完整格式显示所有进程的信息。
-e：显示所有进程，而不仅仅是与当前终端会话相关的进程。
-f：以完整格式显示进程的详细信息，包括进程标识符（PID）、父进程标识符（PPID）、用户、CPU 使用情况等。
|：管道符号，用于将前一个命令的输出作为后一个命令的输入。
grep sshd：这是一个用于文本搜索的命令，它筛选并显示包含 "sshd" 的行。在这种情况下，它用于筛选包含 "sshd" 的进程信息。




```
## ps和grep

```
当涉及到 ps 和 grep 这两个命令时，以下是一些额外的信息和注意事项：

ps 命令：ps（Process Status）是一个用于查看当前运行进程的命令。它提供了对进程的详细信息的访问，如进程标识符（PID）、父进程标识符（PPID）、进程状态、CPU 使用情况等。

常用选项：
-e：显示所有进程，而不仅仅是与当前终端会话相关的进程。
-f：以完整格式显示进程的详细信息。
-u <username>：显示特定用户的进程信息。
-p <pid>：显示特定进程标识符的进程信息。
管道符号 |：管道符号 | 用于将一个命令的输出作为另一个命令的输入。这使得可以通过将多个命令链接在一起来实现更复杂的操作，例如通过筛选、过滤或转换数据。

grep 命令：grep（Global Regular Expression Print）是一个用于文本搜索的强大命令。它根据提供的模式搜索文件或命令输出，并显示包含匹配模式的行。

常用选项：
-i：忽略大小写。
-v：反转匹配，只显示不匹配模式的行。
-r：递归搜索，用于在目录及其子目录中搜索文件内容。
grep 使用正则表达式来指定搜索模式，这使得可以进行更灵活和精确的搜索。

综合应用：通过结合 ps 和 grep 命令，可以搜索特定进程或与特定关键词相关的进程。例如，ps -ef | grep sshd 将显示与 SSH 服务（sshd）相关的进程列表。

在上述示例中，ps -ef 显示所有进程的详细信息，然后通过管道 | 将输出传递给 grep sshd，以筛选包含 "sshd" 的行。


```
## 涉及网络和端口

```
当涉及到网络连接和端口的相关命令时，以下是一些额外的信息和注意事项：

网络连接状态：在计算机网络中，网络连接是指两台计算机之间建立的通信通道。这些连接可以是基于不同的网络协议（如TCP、UDP等）和使用不同的端口号进行标识。

netstat 命令：netstat（Network Statistics）是一个用于查看网络连接和统计信息的命令。它提供了对网络连接、路由表、接口统计和多播成员等的访问。

常用选项：
-a：显示所有连接（包括监听和非监听状态）。
-n：禁用域名解析，以使用IP地址而不是主机名。
-t：仅显示TCP连接。
-u：仅显示UDP连接。
-p：显示与每个连接关联的进程标识符（PID）和进程名称。
ss 命令：ss（Socket Statistics）是一个用于查看套接字统计信息的命令，它是 netstat 命令的替代品。与 netstat 相比，ss 可以提供更快速和更详细的套接字信息。

常用选项：
-a：显示所有连接（包括监听和非监听状态）。
-n：禁用域名解析，以使用IP地址而不是主机名。
-t：仅显示TCP连接。
-u：仅显示UDP连接。
-l：仅显示监听（listening）的套接字。
-p：显示与每个连接关联的进程标识符（PID）和进程名称。
grep 命令：grep（Global Regular Expression Print）是一个用于文本搜索的强大命令，可以与 netstat 或 ss 命令结合使用，以筛选包含特定关键词的行。

常用选项：
-i：忽略大小写。
-v：反转匹配，只显示不匹配模式的行。

```
##  iptable  -vnL

```
iptables -vnL 是一个命令示例，用于列出当前的 IP 表规则和计数器。

解释该命令的各个部分如下：

iptables：这是一个用于管理 Linux 内核网络过滤规则的命令行工具。
-vnL：这是 iptables 命令的选项，用于指定要显示的规则和计数器的信息。
-v：显示详细的规则信息，包括数据包和字节的计数器。
-n：禁用域名解析，以使用 IP 地址而不是主机名。
-L：列出规则，可以选择显示不同的表（如 filter、nat、mangle 等）。
当您运行 iptables -vnL 命令时，它将显示当前生效的 IP 表规则和相关的计数器信息。这些规则用于控制网络流量的传入和传出，允许或阻止特定的数据包。


```

```
这个命令看起来是在以 root 权限执行一个命令。具体命令是 /sbin/ntpdate ntp.ksyun.cn 1>/dev/null 2>/dev/null。

解释一下这个命令的含义：

/sbin/ntpdate: 这是一个用于同步系统时间的命令。ntpdate 命令用于从 NTP（Network Time Protocol）服务器获取准确的时间并将系统时间进行调整。
ntp.ksyun.cn: 这是一个 NTP 服务器的地址，可能是某个云服务供应商的服务器。
1>/dev/null: 这部分是将标准输出（STDOUT）重定向到 /dev/null，/dev/null 是一个特殊的设备文件，它会将所有写入的数据丢弃，相当于将输出丢弃，不显示在终端上。
2>/dev/null: 这部分是将标准错误输出（STDERR）重定向到 /dev/null，同样将错误输出丢弃，不显示在终端上。
综合起来，这个命令的目的是以 root 权限执行 /sbin/ntpdate 命令，从 ntp.ksyun.cn 服务器获取准确的时间，并将输出和错误输出都丢弃，不在终端上显示。

通常情况下，这种命令可能是在系统启动过程中或者定期任务中执行，用于同步系统时间，确保系统时间准确。

```


```
#refused connect from 100.66.32.249          refused connect from 127.0.0.1
"refused connect from 100.66.32.249" 和 "refused connect from 127.0.0.1" 是两条连接被拒绝的消息。

这些消息通常出现在网络连接日志中，用于指示某个主机或 IP 地址的连接被系统拒绝。连接被拒绝的原因可能是以下几种情况之一：

访问控制列表（ACL）：系统可能配置了访问控制列表，限制了从指定 IP 地址或主机进行的连接。

防火墙规则：防火墙可能阻止了从指定 IP 地址或主机进行的连接。

服务未运行或未监听：如果连接尝试访问的服务未运行或未在指定的 IP 地址或端口上进行监听，连接将被拒绝。

连接被拒绝是一种安全机制，旨在保护系统免受未经授权的访问。如果您有特定的需求或需要进一步了解为什么连接被拒绝，您可以考虑检查相关的网络配置、防火墙规则或访问控制列表，并确保目标服务正在运行并监听正确的 IP 地址和端口。
```

