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
1. 使用yum/apt 包管理工具，在线安装redis


2. 本地安装redis
- 在官网下载redis安装包

打开官网下载地址 https://redis.io/download，选择最新版进行下载。

- 解压redis安装包到/usr/local/目录下
> tar -zxf redis-5.0.7.tar.gz -C /usr/local/

- 检查是否已安装redis
> rpm -qa | grep redis
  如果有，使用rpm -e --nodeps redis删除已安装的redis

- 进入redis安装目录，/usr/local/redis.5.0.7/，输入`make`命令编译对redis文件进行编译

如果报错gcc命令未找到，这是因为系统没有安装gcc编译器的原因。

使用命令`yum install -y gcc g++ gcc-c++ make`安装gcc和gcc-c++编译器即可。

如果还有其他报错，参考博客 https://www.cnblogs.com/liu2-/p/6914159.html

- 编译完成之后，可以看到redis-5.0.7文件夹下出现src和conf等文件夹。

进入src目录，执行make install进行redis安装。

- 



# 参考资料
- [Linux安装redis和部署](https://www.cnblogs.com/haoliyou/p/8716624.html)

- [Linux下安装redis报错信息](https://www.cnblogs.com/liu2-/p/6914159.html)

- [CentOS 7系统离线安装gcc，gcc-c++](https://blog.csdn.net/White_Black007/article/details/81357234)

- [Jedis的基本使用](https://blog.csdn.net/weixin_40288381/article/details/88926592)
