[pixiv: 025]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/25.png'
# centos7设置固定Ip方法
IP地址的设置一般是指设置某个网卡端口的IP，所以想设置IP，首先需要弄清楚是要为哪个网卡端口设置IP，CentOS7与一般的Linux系统一样，可以通过ifconfig命令查询当前的网络设置。

![ifconfig结果图](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2501.png)

由上图可知我的系统目前是使用网卡enp0s3访问Internet的，我们要设置的就是enp0s3的IP地址。

弄清楚要设置IP的对象后，需要到etc/sysconfig/network-scripts/下修改enp0s3的配置文件ifcfg-enp0s3。

![ifcfg-enp0s3配置文件](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2502.png)

最后使用vim打开文件并修改如下
![ifcfg-enp0s3配置文件修改项](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2503.png)

如果以上修改完成重启network服务后（service network restart）仍无法联网，可以尝试以下方法：

1. 在ifcfg-enp0s3文件中修改BOOTPROTO为none，IPADDR为192.168.1.9，GATEWAY为192.168.1.1

2. shell中执行命令（nmcli con mod enp0s3 ipv4.dns "114.114.114.114 8.8.8.8"）设置DNS

3. 继续执行命令（nmcli con up enp0s3）是设置生效

4. 重启network服务（service network restart）后，使用ping命令查看联网状态。

使用命令设置DNS并使其生效后，ifcfg-enp0s3文件内容自动修改如下：
![ifcfg-enp0s3配置文件自动修改后的内容](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2504.png)

BOOTPROTO: 系统启动的地址协议，可选参数static(静态地址)，dhcp(DHCP动态地址)，none(不指定地址)，bootp(BOOTP协议)

IPADDR: IP地址

NETMASK: 子网掩码

NM_CONTROLLED: Network Manager参数，yes(使用Network Manager管理网卡)，no(不适用Network Manager管理网卡)

# centos7开放防火墙端口方法
在 Centos 7 或 RHEL 7 或 Fedora 中防火墙由 firewalld 来管理，而不是 iptables。
<hr/>

## 一、firewalld 防火墙
语法命令如下：启用区域端口和协议组合

```shell
firewall-cmd [--zone=<zone>] --add-port=<port>[-<port>]/<protocol> [--timeout=<seconds>]
```

此举将启用端口和协议的组合。
端口可以是一个单独的端口 <port> 或者是一个端口范围 <port>-<port>。
协议可以是 tcp 或 udp。

### 查看 firewalld 状态

```shell
systemctl status firewalld
```

![firewalld状态](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2505.jpg)

### 开启 firewalld(如果查看状态发现已经开启，就不用运行这个命令了)

```shell
systemctl start firewalld
```

![开启firewalld](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2506.jpg)

### 开放端口

```shell
// --permanent 永久生效,没有此参数重启后失效
firewall-cmd --zone=public --add-port=80/tcp --permanent 

firewall-cmd --zone=public --add-port=1000-2000/tcp --permanent 
```

![开放端口](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2507.jpg)

### 重新载入防火墙配置(设置新开放端口后，记得执行这个命令)

```shell
firewall-cmd --reload
```

![重新载入防火墙配置](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2508.jpg)

### 查看端口开放信息

```shell
firewall-cmd --zone=public --query-port=80/tcp
```

![查看端口开放信息](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2509.jpg)

### 删除某个端口开放配置

```shell
firewall-cmd --zone=public --remove-port=80/tcp --permanent
```

## 二、iptables 防火墙

也可以还原传统的管理方式使用 iptables

```shell
systemctl stop firewalld  
systemctl mask firewalld  
```

### 安装 iptables-services

```shell
yum install iptables-services  #在线安装iptables服务
```

### 设置开机启动

```shell
systemctl enable iptables #设置iptables开机自启
```

### 操作命令

```shell
systemctl stop iptables  #停止iptables服务
systemctl start iptables  #启动iptables服务
systemctl restart iptables  #重启iptables服务
systemctl reload iptables   #重新加载iptables服务配置
```

### 保存设置

```shell
service iptables save #保存设置
```

### 开放某个端口 在 /etc/sysconfig/iptables 里添加

```shell
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
```

## 转载文章

- [CentOS7设置固定IP](https://blog.csdn.net/qq_32534441/article/details/86567306)
- [Centos7开放3306端口](https://blog.csdn.net/weiyangdong/article/details/79540217)