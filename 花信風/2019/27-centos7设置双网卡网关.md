[pixiv: 027]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/27.jpg'
# 前言
最近的工作中，一直在配置服务器的网络。所以，也经常会遇到一些网络相关的问题需要解决。当前这个`给双网卡设置网关`，就是其中一个困扰我很久的网络问题。经过一番搜索资料和同事的帮助下，此问题得到解决，现在就问题的出现和解决方案做一个简单的记录，方便以后出现类似地问题时，可以从这篇笔记获取快速解决的方法。
# 问题
项目快要上线了，公司分配了一台centos7的服务器，这台服务器有两张网卡（`em1`,`em2`）。我的工作是，给这两张网卡分别设置两个不同网段的Ip地址，子网掩码以及对应的网关地址。

网卡1`em1`的网络设置是：IP: 192.168.17.15 netmask: 255.255.255.0 gateway: 192.168.17.254
网卡`em2`的网络设置是：IP: 192.168.25.201 netmask: 255.255.255.0 gateway: 192.168.25.1

我通过修改`/etc/sysconfig/network-scripts`目录下的`ifcfg-em1`和`ifcfg-em2`配置文件，已经按照上面的网络设置修改了配置文件的这几项：
```shell
BOOTPROTO="static"
IPADDR=ip
NETMASK=子网掩码
GATEWAY=网关
```
并且，通过`service network restart`重启了网络服务。
通过`route -n`命令查看路由表，也已经看到了两项网关设置。

但是，当我使用`ping`命令测试网络连通性时发现，只有网卡1`em1`可以正常访问`em1`网关以外的网络，网卡2`em2`只能访问`em2`网卡所在网段的网络，`em2`网关之外的网络都不能访问。
这种情况，就好像网卡2的网关设置失效了，网卡2`em2`的网络只能作为局域网使用。

在经过一番资料查找无果后，咨询了同事得知，linux下不能同时设置两个网关，要想给其他网卡设置路由（冲出局域网），只能通过路由命令来实现。

# 解决方案
首先，网卡1`em1`的网络设置不变，正常设置Ip和网关，网卡1可正常使用网络。而网卡2`em2`，我们通过`ifcfg-em2`配置文件只给它设置ip和子网掩码，网关不要设置！不要设置！不要设置！（重要的事情说三遍）。网卡2`em2`的路由，我们通过路由命令来解决。
### 方法一：
1. 在`/etc/sysconfig/network-script/`目录下创建名为`route- em2`的文件
2. `vim route-em2`编辑此文件
3. 写入`192.168.20.3/32 via 192.168.25.1 ` 
>这句话的含义是：指定网卡2`em2`访问IP地址`192.168.20.3`(192.168.20.x网段的交换机)的路由（下一跳）是IP`192.168.25.1`（192.168.25.x网段的交换机） /32的意思和子网掩码类似，32即255.255.255.255，即只指定访问192.168.20.3的下一跳为192.168.25.1。如果是/24，即255.255.255.0，意思就是访问192.168.20.x这个网段的下一跳都是192.168.25.1。

4. 重启网络`service network restart`生效。

### 方法二：
敲命令写路由，但是如果重启通过命令写的路由会失效。不过，我们可以通过`crontab`定时任务命令，设置在开机时自动运行路由设置的脚本，以此解决这个问题。

1. 在`/etc/sysconfig/network-scripts/`目录下创建脚本`routeSet-em2.sh`
2. `vim routeSet-em2.sh`编辑此文件
3. 写入命令
```shell
#!/bin/bash
ip route add 192.168.20.3/32 via 192.168.25.1
service network restart
```

4. `sh routeSet-em2.sh`执行此脚本，此时网卡2的路由已经设置成功，但是重启服务器后，路由设置会失效。
5. 使用`crontab -e`命令编辑当前用户的定时任务，在其中加入`@reboot /etc/sysconfig/network-scripts/routeSet-em2.sh`
6. 输入命令后，点击esc退出键，然后输入`:wq`键，保存crontab文件并退出。定时任务编辑成功并保存后，出现: `crontab:installing new crontab`，则代表定时任务设定成功。
7. 设置crond定时任务服务开机自启，在 /etc/rc.d/rc.local 脚本中加入 /sbin/service crond start 即可
8. 现在，即使重启服务器，你的路由设置也不会失效了:smile:。

# 参考资料
- [linux 添加静态路由](https://blog.csdn.net/moreorless/article/details/5397427)
- [Centos7:利用crontab定时执行任务](https://www.jianshu.com/p/06c6c802d39e)

