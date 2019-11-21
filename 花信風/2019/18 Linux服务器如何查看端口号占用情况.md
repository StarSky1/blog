[pixiv: 018]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/18.jpg'

# 前言
我的开发环境一直是windows，所以经常是在win环境下调试Java代码的时候，遇到端口占用的情况（比如：1.tomcat启动失败，报端口被占用，2.springboot启动失败，报端口被占用）。这时，我会使用cmd shell命令`netstat -ano | findstr "8080"`查看8080端口被哪个进程占用了，然后kill掉这个进程（具体可参考我的文章：[win下解决端口占用命令](https://hexo.starsky1.cn/#/post/11)）。

但是，当我将开发好的程序打war包或jar包，打算在linux服务器上运行它时，发生端口占用，我却不知道怎么办了，因为我只知道解决win下端口占用问题的命令，但是linux下的命令显然和win的shell命令不相同。于是，我打算写下一篇文章，记录下如何在linux服务器下，解决端口占用问题的命令，方便以后遇到类似问题时，快速解决。经过一番网上搜索资料后，下面开始做记录。

# 查看占用指定端口的进程
`netstat  -anp  |grep   端口号`
![命令执行结果](https://hexo.starsky1.cn/poi/2019/181.png)
从截图中可以看到，服务器的80端口是被pid为12783，叫作nginx: worker的进程占用了（注意：监控状态为LISTEN表示已经被占用）。
执行结果的每一行有两端ip，前一段ip+端口是服务器的，后一段ip+端口则是服务器连接的目标机器，端口占用主要看前一段Ip+端口。

# 查看所有端口的占用情况
`netstat   -nultp`
![命令执行结果](https://hexo.starsky1.cn/poi/2019/182.png)
从图中可以看出，我的服务器的80、22、443端口都被占用了，它们分别对应我的网站、ssh服务、https服务。
此处注意，图中显示的LISTENING并不表示端口被占用，不要和LISTEN混淆，查看具体端口时候，必须要看到tcp，端口号，LISTEN那一行，才表示端口被占用。
# 结束占用端口的进程
`kill -9 进程pid`
此命令是linux下专用的结束进程命令，简单好用。比如，运行`kill -9 12783`可以结束掉占用80端口的nginx worker进程。再运行`ps -ef | grep nginx: worker`或者`netstat -anp | grep 80`，可以发现nginx: worker进程已经不存在了，并且80端口也没有被占用了。

# 重新启动你的程序
结束掉占用端口的进程后，就可以成功部署你的程序啦，端口问题就此解决。

# 参考资料

- [LINUX中如何查看某个端口是否被占用](https://www.cnblogs.com/hindy/p/7249234.html)
- [liunx命令查看大全](https://github.com/jaywcjlove/linux-command)