[pixiv: 005]: # 'https://i.loli.net/2019/06/10/5cfdc5e09873554083.png'

## 查找运行端口号为20100的tomcat程序进程
`
ps -ef | grep 20100
`
 运行结果
![运行结果](https://starsky1.github.io/poi/2019/051.png)


## 通过Pid 停止tomcat进程
`
kill -9 30680
`


## 启动tomcat服务器
`
cd apache-tomcat-7.0.68-20100
sh bin/starup.sh
`
## 实时查看tomcat运行日志
`
tail -f logs/catalina.out
`

> tail命令 用于输入文件中的尾部内容。tail命令默认在屏幕上显示指定文件的末尾10行。如果给定的文件不止一个，则在显示的每个文件前面加一个文件名标题。如果没有指定文件或者文件名为“-”，则读取标准输入。

> 注意：如果表示字节或行数的N值之前有一个”+”号，则从文件开头的第N项开始显示，而不是显示文件的最后N项。N值后面可以有后缀：b表示512，k表示1024，m表示1 048576(1M)。
> 实例
tail file #（显示文件file的最后10行）
tail -n +20 file #（显示文件file的内容，从第20行至文件末尾）
tail -c 10 file #（显示文件file的最后10个字符）

实例
<table>
<tr><th>命令</th><th> 注释 </th></tr>
<tr><td>tail -25 mail.log </td><td> # 显示 mail.log 最后的 25 行 </td></tr>
<tr><td> tail -f mail.log  </td><td> # 等同于--follow=descriptor，根据文件描述符进行追踪，当文件改名或被删除，追踪停止 </td></tr>
<tr><td>tail -F mail.log </td><td> # 等同于--follow=name  --retry，根据文件名进行追踪，并保持重试，即该文件被删除或改名后，如果再次创建相同的文件名，会继续追踪 </td></tr>
</table>

## 分享：大佬开源的linux命令大全&搜索工具
<a href="https://github.com/jaywcjlove/linux-command" target="_blank" class="github-corner">
<svg viewBox=""><!-- kenny wang <wowohoo@qq.com> https://github.com/jaywcjlove --> <title>logo</title> <desc>Linux Command Logo. https://github.com/jaywcjlove</desc>
<path d="M128.3,109.0 C113.8,99.7 119.0,89.6 119.0,89.6 C122.0,82.7 120.5,78.6 120.5,78.6 C119.2,72.0 123.4,76.3 123.4,76.3 C127.3,80.9 125.5,87.3 125.5,87.3 C122.9,97.6 130.6,101.9 134.4,103.2" fill="currentColor" style="transform-origin: 130px 106px;" class="octo-arm"></path>
<path d="M115.0,115.0 C114.9,115.1 118.7,116.5 119.8,115.4 L133.7,101.6 C136.9,99.2 139.9,98.4 142.2,98.6 C133.8,88.0 127.5,74.4 143.8,58.0 C148.5,53.4 154.0,51.2 159.7,51.0 C160.3,49.4 163.2,43.6 171.4,40.1 C171.4,40.1 176.1,42.5 178.8,56.2 C183.1,58.6 187.2,61.8 190.9,65.4 C194.5,69.0 197.7,73.2 200.1,77.6 C213.8,80.2 216.3,84.9 216.3,84.9 C212.7,93.1 206.9,96.0 205.4,96.6 C205.1,102.4 203.0,107.8 198.3,112.5 C181.9,128.9 168.3,122.5 157.7,114.1 C157.9,116.9 156.7,120.9 152.7,124.9 L141.0,136.5 C139.8,137.7 141.6,141.9 141.8,141.8 Z" fill="currentColor" class="octo-body"></path>
</svg>
</a>

<font size="5px"><https://github.com/jaywcjlove/linux-command></font>