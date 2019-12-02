---
layout: post
title: 'Esxi安装CentOS7'
date: 2019-11-23
author: 李童
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: 教程
---

#### 参考链接

[VMware安装最新版CentOS7图文教程](https://blog.csdn.net/q2158798/article/details/80550626)

[VMware安装Centos7超详细过程（图文）](https://blog.csdn.net/babyxue/article/details/80970526)

#### 注意事项

- 默认最小安装，是不带GUI桌面的，可以按照需求决定是否安装GUI界面。
- 网络默认是没打开的，安装的时候记得打开，然后根据需求进行配置。

### 常用命令

```
//查看网络配置
ip addr
//测试联网
ping -c 4 www.baidu.com
//安装图形化界面
yum groupinstall -y "gnome desktop"
//切换到图形化界面
init 5
//切换到命令行模式
init 3 
//查看防火墙状态
firewall-cmd --state
//临时关闭防火墙
systemctl stop firewalld
//禁止开机启动
systemctl disable firewalld
//重启网络服务
service network restart
//重启
reboot 
shutdown -r now  
//关机
halt
shutdown -h now  
```

