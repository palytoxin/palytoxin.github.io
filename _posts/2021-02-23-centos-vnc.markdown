---
layout: post
title:  "CentOS VNC Installation"
date:   2021-02-23
categories: linux
---

# 背景

> 之前在升级OpenSSH的过程中为了防止机器走丢，安装了VNC工具，本文以记录安装过程


# Show me the code.

1、安装VNC Service

`yum install -y tigervnc tigervnc-server`

2、拷贝配置文件

`cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service`

3、修改配置文件

`vim /etc/systemd/system/vncserver@:1.service`

> 修改内容，之前测试旧版本有bug需要修改部分挺多，新版多了vncserver_wrapper命令，只需要将`<USER>`修改为用户名就可以了。

4、添加开机启动

`systemctl enable vncserver@:1.service`

5、设置VNC密码

`vncpasswd`

6、启动VNC服务

`systemctl start vncserver@:1.service`

# 结尾清扫

因为是临时需要vnc，所以需要临时关闭防火墙或者是为防火墙添加规则。
```bash
# 查看vnc端口
ss -tulpn| grep vnc
# 添加规则
firewall-cmd --add-port=5901/tcp
firewall-cmd --add-port=5901/tcp --permanent
```

在CentOS上连接VNC可以安装`vinagre`软件包