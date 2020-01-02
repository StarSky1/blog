[pixiv: 028]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/28.jpg'
# 前言
最近工作中，需要给一些在服务器运行的jar包服务设置开机自启，防止服务器意外宕机重启后，这些服务都不能正常使用。
# 方法
使用`crontab`命令，设置开机自启动任务。
>注意：
1）通过cron.service定时服务来调用可执行文件时，cron是无法加载系统中的相关路径设置的，因此在脚本文件中，相关路径都要转换为绝对路径。尤其jdk的路径应当为其安装路径。
2）在执行start java application流程时，我们应当通过cd命令，对系统运行目录进行切换，以转到相应服务目录下。

### 1、为.sh脚本文件添加权限
1. 对于windows环境下编辑的.sh可执行文件，我们拷贝到linux环境后，在cron.service服务中是没有权限执行该脚本的，但通过手动输入：sh *.sh命令，是可以成功执行该脚本的，因此我们需要为该脚本添加权限，以使其在Linux环境下为可执行文件。
2. 通过命令：`ls –l `，我们可以查看文件的相关属性，下面的test.sh是非可执行文件，文件为灰色。![图1](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/28/2801.png)
3. 通过命令：`chmod 744 test.sh`，我们就可以将`test.sh`转为可执行文件，文件名为绿色，同时x也代表该文件为可执行文件。
![图2](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/28/2802.png)


### 2、编辑crontab定时任务
1. 下面我们以root用户为例，来编辑crontab定时任务，对于root用户，我们可以直接通过命令：`crontab –e`，打开crontab文件编辑器，点击`i`键，就可以对crontab文件进行编辑。
![图3](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/28/2803.png)

2. 输入命令：`@reboot /root/LinuxAutoStartProject_jar/test.sh`
@reboot，指定服务器启动时，cron.service定时任务会在该时间点执行相应的事务。
/root/LinuxAutoStartProject_jar/test.sh，指定了定时任务将要执行的任务，可以是相关Linux命令，也可以是相关可执行的脚本程序。
3. 输入命令后，按`esc`退出键，然后输入`:wq`键，保存crontab文件并退出。定时任务编辑成功并保存后，出现`: crontab:installing new crontab`，则代表定时任务设定成功。
![图4](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/28/2804.png)
### 3、设置`crond`服务开机自启动
设置crond定时任务服务开机自启，在 `/etc/rc.d/rc.local `脚本中加入 `/sbin/service crond start `即可

# 参考资料
- [Centos7:利用crontab定时执行任务](https://www.jianshu.com/p/06c6c802d39e)
