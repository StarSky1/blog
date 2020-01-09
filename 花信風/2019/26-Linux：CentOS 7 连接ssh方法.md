[pixiv: 026]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26.jpg'

## 【转载】(https://www.cnblogs.com/nhdlb/p/10515924.html)
<p>=================以下转自那位博主===================</p>

##### 在虚拟机（Vmware Workstation）下，安装了CentOS7，现在想通过SSH工具连接虚拟机中的CentOS7

1、  首先，要确保CentOS7安装了  openssh-server，在终端中输入  yum list installed | grep openssh-server
![图1](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/01.jpg)
此处显示已经安装了  openssh-server，如果又没任何输出显示表示没有安装  openssh-server，通过输入  yum install openssh-server
![图2](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/02.jpg)
来进行安装openssh-server

2、  找到了  /etc/ssh/  目录下的sshd服务配置文件 sshd_config，用Vim编辑器打开将文件中，关于监听端口、监听地址前的 # 号去除

（备注：博主省去了一些操作方法，我作为菜鸟，还是补充下 vim 进入文本，按“i”开始编辑，编辑好之后按“esc”退到命令模式，按“:wq”保存 并退出）
![图3](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/03.jpg)

然后开启允许远程登录

![图4](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/04.jpg)

最后，开启使用用户名密码来作为连接验证

![图5](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/05.jpg)

保存文件，退出

3、  开启  sshd  服务，输入 sudo service sshd start
![图6](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/06.jpg)

检查  sshd  服务是否已经开启，输入ps -e | grep sshd

![图7](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/07.jpg)

或者输入netstat -an | grep 22  检查  22 号端口是否开启监听

![图8](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/08.jpg)

4、  在Vmware Workstation中，查看CentOS7的属性，发现网络连接方式是采用的  NAT  方式连接的

![图9](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/09.jpg)

5、 在Vmware Workstation中，点击编辑=》虚拟网络编辑器，进入虚拟网络编辑器，查看发现 NAT 模式的连接采用的网络适配器名称为VMnet8

![图10](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/10.jpg)

6、在 windows 主机中，在命令行中输入ipconfig 查看主机IP，找到 VMnet8 的连接信息，此处 ip 为192.168.30.1

![图11](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/11.jpg)

7、在CentOS中，输入ifconfig查看网络连接地址，发现CentOS的网络地址为192.168.112.128

![图12](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/12.jpg)

8、在CentOS中，输入ping 192.168.30.1 测试是否能连通主机，发现可以连通

![图13](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/13.jpg)

9、在主机中，输入 ping 192.168.112.128，测试主机是否能连通CentOS，发现连不通

（备注：我这边也没联通，用了博主的方式也不行，因为不是同一个原因，我的问题在于本机的网络设置中，虚拟机的以太网IP被我设置成固定的了， 其实应该是由路由器分配的，这边的情况是这样...所以将以太网3中TCP/IP 4属性改为自动获取IP地址）

![图14](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/14.jpg)

如果可以连得通，可以直接跳至第12 步

10、在主机，打开网络配置，选择网络适配器 VMnet8 的  TCP/IPv4   的属性，进行一下网络配置

![图15](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/15.jpg)

要求子网掩码、默认网关均和CentOS一致，并将IP地址修改为 192.168.112.1，即保证主机的  IP  和  CentOS  的  IP  在同一网络区段中

11、再在主机中，输入 ping 192.168.112.128，已经可以连接得通了

![图16](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/16.jpg)

12、在SSH工具（此处使用的XShell）中，新建连接，输入  CentOS   的  IP  地址、用户名、密码即可连接成功

![图17](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/17.jpg)

![图18](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/18.jpg)

连接成功

![图19](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/19.jpg)

13、为了免去每次开启 CentOS 时，都要手动开启  sshd 服务，可以将 sshd 服务添加至自启动列表中，输入systemctl enable sshd.service

（备注：我用的是CentOS 6.3 版本老，命令是[root@localhost ~]# chkconfig sshd on  注意[ ...~]表示的是/root路径）

![图20](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/20.jpg)

可以通过输入systemctl list-unit-files | grep sshd，查看是否开启了sshd 服务自启动

![图21](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/26/21.jpg)

文章转至：https://blog.csdn.net/tuntun1120/article/details/65443757

<p>----------------------------------- 作者：怒吼的萝卜 链接：http://www.cnblogs.com/nhdlb/ -----------------------------------</p>
