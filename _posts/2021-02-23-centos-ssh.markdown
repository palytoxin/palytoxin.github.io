---
layout: post
title:  "CentOS SSH Installation"
date:   2021-02-23
categories: linux
---

# 背景

> 某项目上线需要进行安全检测，服务器openssh-server版本是7.4p1，被检测出安全漏洞禁止上线。本文记录如何自己编译新版openssh替换系统自带的版本。
> 现在的服务器都很难接触到物理机，所以需要给自己开个后门(VNC/telnet)，防止ssh断开后无法连接服务器。


# Show me the code.

```bash
# 停止系统预装sshd服务
systemctl stop sshd
systemctl disable sshd
# 卸载系统预装openssh
yum remove -y openssh-server
# 安装编译工具
yum install -y zlib-devel pam-devel gcc openssl-devel
# 备份原有ssh配置
mv /etc/ssh /etc/ssh_bak
# 从openbsd官网下载最新版本openssh
wget https://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-8.4p1.tar.gz
tar xvf openssh-8.4p1.tar.gz
cd openssh-8.4p1.tar.gz

# 编译安装
./configure --prefix=/usr --sysconfdir=/etc/ssh --with-pam --with-zlib=/usr/local/zlib --with-ssl-dir=/usr/local/openssl --with-md5-passwords --without-hardening
make
make install

# 还原sshd配置
cp sshd_config /etc/ssh/sshd_config

# 因默认情况下无systemd启动服务，才用redhat服务开启登录配置
cp ./contrib/redhat/sshd.init /etc/init.d/sshd84
chmod +x /etc/init.d/sshd84
chkconfig --add sshd84
service sshd84 restart
```

# 结尾清扫

> 因为默认CentOS开启了SELINUX，编译安装之后ssh端口转发被selinux禁止，因为某些调试原因经常需要用到端口转发。
> 开启之前需要使用端口转发使audit记录失败。

```bash
ausearch -c 'sshd' --raw | audit2allow -M my-sshd
semodule -i my-sshd.pp
```