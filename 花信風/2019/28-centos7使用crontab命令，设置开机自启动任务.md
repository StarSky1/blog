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

### 4、后台运行jar包的脚本文件
```shell
#!/bin/bash
WORK_DIR="/root/app/app_jar"
JAR_NAME="app.jar"
MY_JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64/jre/bin/java"
HOST_IP="127.0.0.1"

pid=`ps -ef | grep $JAR_NAME | grep -v grep |awk '{print $2}'`
echo "===>pid:$pid"
if test -n "$pid"
then
ps -ef|grep $JAR_NAME|grep -v grep|awk '{print $2}'|grep $pid > /dev/null
if test $? -eq 0
then
echo "The process is running !\n"
exit 3
fi
fi

#Check process is existence
if [ ! -f $WORK_DIR/$JAR_NAME ] ; then
   echo "Error: Can not find the file $WORK_DIR/$JAR_NAME,shik next shell"
   exit 3
else
   echo "$WORK_DIR/$JAR_NAME exist,go next"
fi

#start jar
echo "start java application"
cd $WORK_DIR
nohup $MY_JAVA_HOME -Djava.library.path=lib -Djava.rmi.server.hostname=$HOST_IP -Xms128m -Xms256m -XX:+UseParallelOldGC -XX:ParallelGCThreads=2 -jar $JAR_NAME > out.log 2>&1 &
```

### 5、nohup命令
nohup 命令运行由 Command参数和任何相关的 Arg参数指定的命令，忽略所有挂断（SIGHUP）信号。在注销后使用 nohup 命令运行后台中的程序。要运行后台中的 nohup 命令，添加 & （ 表示“and”的符号）到命令的尾部。

nohup 是 no hang up 的缩写，就是不挂断的意思。

nohup命令：如果你正在运行一个进程，而且你觉得在退出帐户时该进程还不会结束，那么可以使用nohup命令。该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。

在缺省情况下该作业的所有输出都被重定向到一个名为nohup.out的文件中。
<br/>
用法举例： nohup command > myout.file 2>&1 &   

在上面的例子中，0 – stdin (standard input)，1 – stdout (standard output)，2 – stderr (standard error) ;
2>&1是将标准错误（2）重定向到标准输出（&1），标准输出（&1）再被重定向输入到myout.file文件中。

# 参考资料
- [Centos7:利用crontab定时执行任务](https://www.jianshu.com/p/06c6c802d39e)
- [nohup命令详解](https://www.cnblogs.com/jinxiao-pu/p/9131057.html)