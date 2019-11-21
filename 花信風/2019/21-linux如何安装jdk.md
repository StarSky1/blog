[pixiv: 021]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/21.jpg'
## 第一步，下载linux版本的JDK
1.打开oracle官网JDK8下载页，地址：[here](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

2.选择Linux x64版本的jdk-8u231-linux-x64.tar.gz压缩包，点击下载。发现需要登录oracle账号，我没有oracle账号，所以我选择注册一个（注册其实很简单，1分钟就注册好了）。

另外我在百度云盘上存了一份JDK8 linux版本压缩包，你也可以在百度云盘上下载。
```
链接: https://pan.baidu.com/s/1dTrbmCyyJULI6h6AzfMGmw 提取码: xrmh 复制这段内容后打开百度网盘手机App，操作更方便哦
```

下载完成后
（1）在/usr/local/目录下创建一个java目录
```
cd /usr/local/
mkdir java
```
（2）将.tar.gz包拷贝到/usr/local/java目录下，并解压
执行 `tar -zvxf jdk-8u231-linux-x64.tar.gz`

(3) centos7版本的系统会自带jdk，如果你也是centos的系统，请执行下面的命令，其他系统请自行搜索如何卸载Jdk。

步骤一：查询系统是否已安装jdk
```shell
rpm -qa|grep java  查看系统中是否安装了jdk。
输出类似：java-1.8.0-openjdk-1.8.0.131-11.b12.el7.x86_64
```
步骤二：卸载已安装的jdk
```shell
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.131-11.b12.el7.x86_64
```
步骤三：验证一下还有没有jdk
```shell
rpm -qa|grep java
java -version
#如果输出为空，则证明已经卸载了自带的Jdk
```
(4) 安装刚才下载好的JDK8

步骤一：查看我们解压好的JDK8所在目录
```shell
cd /usr/local/java  #切换到我们刚才解压jdk-8u231的目录。
ls  #可以查看到我们解压的Jdk8目录
输出类似：jdk-8u231-linux-x64
```
步骤二：编辑/etc/profile文件，给Java设置环境变量
```shell
cp /etc/profile /etc/profile-bak #备份环境变量文件
vim /etc/profile  #编辑环境变量文件
在profile文件末尾添加
export JAVA_HOME=/usr/local/java/jdk-8u231-linux-x64
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

```
步骤三：重新编译/etc/profile文件
```shell
执行 source profile #重新编译系统环境变量
执行 java -version
输出类似：java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b12, mixed mode)
说明JDK8已经安装成功
```

## 参考文章
- [Centos7-卸载自带的jdk 安装jdk8](https://www.cnblogs.com/happyflyingpig/p/8068020.html)


