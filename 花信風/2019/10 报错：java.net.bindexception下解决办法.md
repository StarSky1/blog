[pixiv: 006]: # 'https://i.loli.net/2019/06/10/5cfdc64b0563774446.jpg'

## 根据端口号查找进程
1. windows下cmd打开命令行，运行这个命令
`netstat -ano | findstr "8082"`
![执行结果](https://starsky1.github.io/poi/2019/061.png)
2. 输入tasklist | findstr "10808"   10808是这个进程的Pid
`tasklist | findstr "10808"`
![结果](https://starsky1.github.io/poi/2019/062.png)
 WTF!  QQ占用了8082？？？ 
3. 输入taskkill /im qq.exe /f 终止这个进程
`taskkill /im qq.exe /f`
![结果](https://starsky1.github.io/poi/2019/063.png)
4. 重新启动Springboot项目，查看运行是否成功
![结果](https://starsky1.github.io/poi/2019/064.png)
好了，可以继续开发了~
![](https://starsky1.github.io/poi/2019/065.png)
----
## 参考资料
- [报错：java.net.bindexception: address already in use: jvm_bind:8080](https://www.cnblogs.com/htys/p/3274795.html)
- [windows使用命令行杀进程](https://www.cnblogs.com/shindo/p/5959329.html)