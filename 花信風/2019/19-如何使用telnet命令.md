[pixiv: 019]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/19.jpg'
# 为什么写这篇文章
最近一段时间，我的工作都和部署服务有关，Java服务、web服务等等。公司给我搭建了几台服务器，系统环境是linux，系统版本是centos7。因为服务器上没有安装开发环境，我需要自己去安装一些开发环境，比如JDK、Mysql数据库等。在我安装开发环境以及部署服务的过程中，经常需要测试部署的服务是否在运行，或者安装的开发环境是否在运行。这时，我需要测试我的服务或开发环境使用的端口是否可访问，比如websocket服务使用的端口、mysql数据库使用的3306端口是否可访问。总所周知，ping命令可以测试ip的连通性，但是对于端口而言，ping命令无能为力。于是，我找到了telnet命令，它可以测试端口的连通性。这篇文章就是对如何使用telnet命令测试端口可访问性的学习笔记。
# windows环境如何使用telnet命令
1. 首先，win+r键打开运行，输入cmd打开命令行程序。然后输入`telnet 127.0.0.1 8080`（第一个参数Ip，第二个参数端口，不输入默认是23端口）。如果提示类似
![图1](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/1802.png)
那么说明你的windows系统还没有开启telnet功能。
2. 可以在控制面板-程序和功能-启用或关闭windows功能-勾选“Telnet客户端”，点击确定。然后，在命令行中就可以使用telnet命令。
类似
![图2](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/1801.png)
3. 现在重新打开一个命令行程序，输入`telnet 127.0.0.1 3306`。如果提示类似这样的黑框，那么说明这个端口是可访问的。
![图3](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/1803.png)
4. 输入`telnet 127.0.0.1 8080`，如果提示类似8080端口连接失败，那么表示8080端口是不可访问的，未开放的。
![图4](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/1804.png)
5. telnet可以用于远程登录主机，对远程主机进行管理。

telnet远程登录通信的时候，采用的是明文。如果有人使用嗅探工具抓包，在你用telnet通信的时候，抓取你的信息数据包，被抓取的数据包为明文。因此telnet通信是不安全的，很多linux服务器都不开放telnet服务，而采用SSH服务。Windows主机一般来说也是默认不开放telnet服务的，现在windows7等系统甚至连telnet客户端默认都没有安装。

使用telnet ip ，可以开始一个telnet会话，输入用户名和密码来登录远程服务器，以命令行的方式远程管理计算机。需要注意的是，远程机器必须启动telnet服务。

# Linux环境下如何使用telnet命令
1. 检查是否已经安装telnet客户端
在linux环境下，要使用telnet命令，你需要安装一个telnet软件包。
安装之前，检测是否已安装此软件包，centos下命令如下：
```
[root@localhost ~]# rpm -qa | grep telnet
telnet-0.17-48.el6.x86_64
```
如果已经安装，可以直接使用。或者，使用`rpm -e --nodeps telnet-0.17-48.el6.x86_64`卸载掉。
2. 查看yum仓库中的telnet安装包名字
```
[root@localhost ~]# yum list telnet* 
Loaded plugins: fastestmirror
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
Determining fastest mirrors
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Available Packages
telnet.x86_64                                                                          1:0.17-64.el7                                                                    base
telnet-server.x86_64
```
3. 安装telnet客户端
```
[root@localhost ~]# yum install -y telnet.x86_64
```
4. 测试telnet客户端是否安装成功
```
[root@localhost ~]# telnet 127.0.0.1 9501
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
```





# 参考文章
- [telnet命令](https://blog.csdn.net/qiushisoftware/article/details/92811763)
- [linux下安装telnet](https://www.jianshu.com/p/47b17927f23a)

