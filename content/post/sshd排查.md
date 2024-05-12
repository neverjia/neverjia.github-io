---
title: "sshd相关错误"
slug: "sshd相关错误"
date: "2024-05-11"
created: "2024-05-11"
tags: ["sshd"]
---
sshd远程不上
## 排查思路
```
①主机内有没有限源。
②/etc/hosts.deny
③/etc/hosts.allow
④iptable有没有相关的策略。
⑤服务是否正常运行。
```



### 详细步骤

```
①ssh无法登录，先确定网没有问题，更换VPC以后一定要重启主机或者重启网卡、安全组、acl没有限制；
EIP是fix ip的nat地址。fix ip都不对了，nat怎么可能还生效呢。nat记录路由是新的fix ip和eip。
②堡垒机无法登录的话，可以通过其他主机登录看看，如果也无法登录的话 可以在其他主机执行ssh -vvvv，观察ssh的type id；
③并且结合目标主机的secure日志判断问题所在。

ip -o 4 a
systemctl restart network
```






当遇到SSH无法远程登录的问题时

1. **网络连通性检查**：
   - 使用`ping`命令测试从客户端到服务器的网络连通性，确认网络没有问题。
   - 检查是否有路由问题或DNS解析问题，确保能正确解析服务器IP。

2. **SSH服务状态**：
   - 在服务器上运行`systemctl status sshd`或`service ssh status`（取决于系统使用的init系统），确认SSH服务是否正在运行。如果未运行，尝试启动服务并检查日志文件了解失败原因。

3. **防火墙设置**：
   - 确认服务器的防火墙规则是否允许SSH连接（默认端口22）。可以使用`ufw status`或`iptables -L`查看。
   - 如果使用云服务器，还需检查云服务商的安全组规则，确保入站流量允许SSH端口。

4. **SSH配置**：
   - 检查`/etc/ssh/sshd_config`配置文件，确认没有禁止SSH登录的配置，如`PermitRootLogin`设置（如果尝试以root身份登录）。
   - 确认监听地址是否正确，如`ListenAddress`设置。
   - 若有修改，记得重启SSH服务使配置生效。

5. **端口监听**：
   - 使用`netstat -tulnp | grep ssh`或`ss -tuln | grep ssh`检查SSH服务是否在预期的端口上监听。

6. **密钥与认证**：
   - 确认是否使用了正确的认证方式（密码或密钥对），以及密钥是否正确配置在`~/.ssh/authorized_keys`。
   - 如果使用密钥认证，检查客户端的私钥权限和是否正确使用。

7. **SELinux或AppArmor**：
   - 如果服务器启用了SELinux或AppArmor，检查这些安全策略是否限制了SSH访问。

8. **系统日志**：
   - 查看系统日志，如`/var/log/auth.log`或`/var/log/secure`，寻找SSH登录失败的相关错误信息，这往往能直接指出问题所在。

9. **主机名解析**：
   - 确认`/etc/hosts`文件没有错误地映射了SSH服务器的IP地址。

10. **用户权限与限制**：
    - 确认用户账户是否被锁定，或是否有正确的shell设置（通常在`/etc/passwd`文件中查看）。







>  其他备注：https://wiki.archlinux.org/title/GNOME/Keyring
> 这个pam不会影响ssh；它的作用只是gnome group，并且只适用于密钥环的校验。确实文件应该是一直在打，或者每次涉及到输入密码的时候就会报错，安装 libgnome-keyring就有这个文件了。





```
正在查看的是btmp日志文件的开始部分，使用了lastb命令来实现。
#lastb
lastb: 命令用来查看btmp文件的内容。btmp是Linux系统中的一个日志文件，专门记录所有不成功的登录尝试，包括SSH、telnet等登录失败的情况。这个文件可以帮助系统管理员识别潜在的安全威胁或暴力破解尝试。
```

