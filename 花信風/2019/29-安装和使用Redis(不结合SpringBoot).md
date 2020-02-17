[pixiv: 029]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29.jpg'

# 什么是Redis
Redis（全称：Remote Dictionary Server 远程字典服务）是一款使用C语言编写的基于内存的key-value数据库。它支持网络访问，是可进行持久化的日志型数据库，并提供多种语言的API。--来自百度百科。

通常，在项目中，redis是作为缓存使用的。这次我使用redis的目的，也是将它作为缓存来使用的。

# 为什么使用redis
本次工作，我的任务是监控一些设备在48小时内是否发生异常，以48小时为时间间隔，周期性进行检查。也就是说，每个设备每隔48小时需要判断一次是否有异常。

考虑到设备数量较多，放在Jvm内存中存储害怕发生out of memory。于是决定将记录每个设备检查时间的时间戳存入redis中。另外，也将所有设备异常记录以hash类型的方式存入redis中，设备检查时间则是使用redis的string类型保存。

# windows下如何使用redis
1. 下载windows版本的redis

由于在redis官网只能下载linux版本的redis安装包，所以需要在github上下载windows版本的redis。
地址是：https://github.com/MicrosoftArchive/redis/releases
下载最新版本的zip包就好了。

2. 安装redis

将redis的压缩包解压到指定文件夹（比如redis-3.2/），安装就完成了。

3. 运行redis服务

- 打开cmd命令行程序（快捷键 开始键+R，输入cmd并回车）
- cd到redis安装的目录（我的是D:\redis-3.2.100），输入命令`redis-server.exe redis.windows.conf`即可启动redis服务，效果如下。
![启动redis数据库](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29/2901.png)

4. 使用redis desktop manager软件查看redis数据库

输入ip: localhost,port: 6379，密码为空，连接redis数据库，效果如图。
![查看redis数据库](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29/2902.png)

# linux下使用redis数据库
[123注释]: # '1. 使用yum/apt 包管理工具，在线安装redis'

1. 本地安装redis
- 在官网下载redis安装包

打开官网下载地址 https://redis.io/download，选择最新版进行下载。

- 使用winSCP远程工具，拷贝redis安装包到Linux服务器，然后解压redis安装包到/usr/local/目录下
> tar -zxf redis-5.0.7.tar.gz -C /usr/local/ \
或者直接在Linux服务器上通过命令 wget 安装包下载地址 下载安装包

- 检查是否已安装redis
> rpm -qa | grep redis
  如果有，使用rpm -e --nodeps redis删除已安装的redis

- 进入redis安装目录，/usr/local/redis.5.0.7/，输入`make`命令编译对redis文件进行编译

如果报错gcc命令未找到，这是因为系统没有安装gcc编译器的原因。

使用命令`yum install -y gcc g++ gcc-c++ make`安装gcc和gcc-c++编译器即可。

离线安装gcc和gcc-c++编译器，请参考：[CentOS 7系统离线安装gcc，gcc-c++](https://blog.csdn.net/White_Black007/article/details/81357234)

如果还有其他报错，参考博客 https://www.cnblogs.com/liu2-/p/6914159.html

- 编译完成之后，可以看到redis-5.0.7文件夹下出现src文件夹。

进入src目录，执行make install进行redis安装。
安装结果如图所示：![make install结果](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29/2903.png)

 - 第三步：部署

1.为了方便管理，将Redis文件中的conf配置文件和常用命令移动到统一文件中

1）、创建bin和conf.d文件夹

如图示: ![创建目录](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29/2904.png)

2）、回到刚刚安装目录，找到redis.conf，将其复制移动到 /usr/local/redis/conf.d 下
执行命令如下：
> cp redis.conf /usr/local/redis/conf.d/

进入src目录，复制 mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-server到/usr/local/redis/bin/

>执行命令 ：cp mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-server /usr/local/redis/bin/

效果如图所示：![复制后bin和conf.d的目录内容](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29/2905.png)

3)、执行redis-server 启动redis

![启动redis](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29/2906.png)

4)、修改./conf.d/redis.conf配置文件,设置绑定ip（注：该步骤如果不需要可省略）

![设置绑定ip](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29/2907.png)
如需要，可将上图绑定ip改为指定ip。

5)、设置后台启动redis

首先编辑redis.conf文件，将daemonize属性改为yes（表明需要在后台运行）

                   cd conf.d/
                   vim redis.conf
![修改配置文件](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29/2908.png)

将no修改为yes
再次启动redis服务，并指定启动服务配置文件
> redis-server /usr/local/redis/etc/redis.conf

![启动成功图](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29/2909.png)

配置后台运行成功。

6)、后台模式下停止redis
> 执行命令 ./bin/redis-cli shutdown \
等待1分钟，使用命令netstat -anp | grep 6379 查看redis进程已终止

效果如图: ![redis停止图](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29/2910.png)

- 第四步：配置redis服务可远程访问

1)、编辑./conf.d/redis.conf配置文件,注释掉bind 127.0.0.1可以使所有的ip访问redis，关闭保护模式（protected-mode no）

如图所示：![修改配置](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/29/2911.png)

2)、重新启动redis服务，并指定刚才修改的配置文件
> redis-server /usr/local/redis/etc/redis.conf 即可

另外，注意防火墙是否开放了6379端口，如果没开放6379端口，远程访问redis可能会失败。

# Java中使用redis服务
参考我的redis代码笔记：[代码笔记](https://gitee.com/StarsSky/JavaNotes/tree/master/redis)

重点看 RedisPool、 RedisPoolUtil 这两个类。

# 参考资料
- [Linux安装redis和部署](https://www.cnblogs.com/haoliyou/p/8716624.html)

- [Linux下安装redis报错信息](https://www.cnblogs.com/liu2-/p/6914159.html)

- [CentOS 7系统离线安装gcc，gcc-c++](https://blog.csdn.net/White_Black007/article/details/81357234)

- [redis开启远程访问](https://www.cnblogs.com/Gyoung/p/6678702.html)

- [Jedis的基本使用](https://blog.csdn.net/weixin_40288381/article/details/88926592)
