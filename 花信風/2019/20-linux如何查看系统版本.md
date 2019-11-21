[pixiv: 020]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/20.jpg'
## 查看linux操作系统版本
### 转载自：https://blog.csdn.net/wanchaopeng/article/details/83818323

1. 查看内核版本的命令

```shell
[root@card-db02 ~]# cat /proc/version 
Linux version 3.10.0-693.2.2.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) ) #1 SMP Tue Sep 12 22:26:13 UTC 2017
[root@card-db02 ~]# uname -a
Linux card-db02 3.10.0-693.2.2.el7.x86_64 #1 SMP Tue Sep 12 22:26:13 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
[root@card-db02 ~]# uname -r
3.10.0-693.2.2.el7.x86_64
```

2. 查看Linux 版本

1） 执行`lsb_release -a` ,即可列出所有版本信息,例如:
```shell
[root@card-db02 ~]# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.4.1708 (Core) 
Release:	7.4.1708
Codename:	Core
```
注:这个命令适用于所有的linux，包括Redhat、SuSE、Debian等发行版。

2）如果是centos或基于redhat的linux，执行`cat /etc/redhat-release` ,例如如下:
```shell
[root@card-db02 ~]# cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core)
```
3）Linux 常用的查看当前系统版本的命令 `cat /etc/os-release`
```shell
root@iZwz9b9lhklwm0avfcq1ywZ:/etc/docker# cat /etc/os-release 
NAME="Ubuntu"
VERSION="16.04.3 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04.3 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
VERSION_CODENAME=xenial
UBUNTU_CODENAME=xenial
root@iZwz9b9lhklwm0avfcq1ywZ:/etc/docker# 
```
