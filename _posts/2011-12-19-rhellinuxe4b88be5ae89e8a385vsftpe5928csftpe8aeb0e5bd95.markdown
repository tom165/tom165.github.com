---
author: tom165
comments: true
date: 2011-12-19 14:14:47+00:00
layout: post
slug: rhellinux%e4%b8%8b%e5%ae%89%e8%a3%85vsftp%e5%92%8csftp%e8%ae%b0%e5%bd%95
title: RHELlinux下安装vsftp和sftp记录
wordpress_id: 127
categories:
- 网络技术
---

因需要，要在RHEL linux 5.5 下安装FTP服务器，用于WLAN下的FTP下载测试，FTP目录下建立wifi_down,wifi_up两个目录，分别用于下载与上传测试，查了资料，还算顺利完成，简要记录如下：

<!-- more -->

1、下载vsftp的RPM软件包到服务器里
wget ftp://XXX.XXX.XXX.XXX/vsftp2.0.5.rpm

2、安装vsftp软件包

rpm -ivh vsftpd*.rpm

3、配置vsftp

修改vsftpd.conf 配置文件

vi /etc/vsftpd/vsftpd.conf

xferlog_enable=YES
xferlog_file=/var/log/vsftpd.log
log_ftp_protocol=YES ---开启详细LOG

max_clients=500 --最大500用户
max_per_ip=5 --每Ip最多5个用户
local_max_rate=8192000 ---最大下载8M

chroot_local_user=YES ---用户锁定在自身目录
4、创建FTP使用的用户：
adduser -d /var/ftp/wifi_test -s/sbin/nologin wlan ---创建FTP用户名，使用户不能登录执行系统命令
passwd wlan  ---修改密码

5、在wifi_test下 创建wifi_up,wifi_down两个目录

mkdir wifi_up
mkdir wifi_down
修改目录权限使wifi_down可读（只能下载），wifi_up可读写（可以下载上传）

[root@localhost ~]# ll /var/ftp/wifi_test

dr-xr-xr-x 2 wlan ftp 4096 Dec 9 10:47 wifi_down
drwxr-xr-x 2 wlan ftp 4096 Dec 9 12:28 wifi_up

6、用dd命令在wifi_down下生成特定大小的文件(50M，200M，1G，5G，10G)，便于下载测试

dd if=/dev/zero of=50M bs=1M count=50
dd if=/dev/zero of=200M bs=1M count=200
dd if=/dev/zero of=1G bs=1M count=1024
dd if=/dev/zero of=5G bs=1M count=5012
dd if=/dev/zero of=10G bs=1M count=10240
7、在iptables防火墙上开放ftp服务，
(1 )编辑/etc/sysconfig/iptables-config

##############add by tom165#########
IPTABLES_MODULES="ip_conntrack_ftp"
IPTABLES_MODULES="ip_nat_ftp"
请一定注意两行内容的位置关系不要搞反了。如果将”ip_nat_ftp”放到前面是加载不到的。如果你的ftp服务是过路由或者防火墙（即内网映射方式一定需要此模块）。以上等同于在加载iptables之前运行modprobe命令加载”ip_nat_ftp”和”ip_conntrack_ftp”模块。

(2)编辑/etc/sysconfig/iptables文件添加如下两行：

-A RH-Firewall-1-INPUT -p tcp -m state –-state NEW -m tcp –-dport 21 -j ACCEPT
-A RH-Firewall-1-INPUT -p tcp -m state –-state NEW -m tcp –-sport 21 -j ACCEPT

(3)、检查iptables文件是否存在以下行（默认是有的），如没有则添加；
-A RH-Firewall-1-INPUT -m state –state ESTABLISHED,RELATED -j ACCEPT

8、
重启iptables服务
# service iptables restart

重启iptables服务
# service vsftpd restart
9、开启sftp服务
编辑 /etc/sshd/sshd_config的配置:
#Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp ---使用内置的sftp server
Match User wlan           ---匹配到wlan目录则锁死目录
ChrootDirectory /var/ftp/wlan_test
