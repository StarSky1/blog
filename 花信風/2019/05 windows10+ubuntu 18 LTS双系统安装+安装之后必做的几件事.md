[pixiv: 001]: # 'https://i.loli.net/2019/06/10/5cfdc3c132d7836717.jpg'

#  windows10+ubuntu 18 LTS双系统安装+安装之后必做的几件事

---


# 一、安装ubuntu 18 LTS

## 1.分出一个空白卷
首先，进入windows10系统，右击我的电脑，依次点击 管理->磁盘管理，进入磁盘管理。选择一个容量较大空余空间较多的分区，右键点击压缩卷，按照你的需要分出一个指定大小的空白卷，我这里选择分出了60G用来安装ubuntu系统。注意，这里压缩一个卷就行了，不用格式化。
## 2.制作ubuntu系统安装U盘启动器
使用一种制作U盘启动器的工具，将下载好的ubuntu镜像文件放入U盘中。我这里推荐使用rufus这个小巧强大的U盘启动器制作工具。
## 3.重启电脑，进bios设置boot priority为U盘优先启动
我的电脑开机进bios的方法是按del，电脑不知道怎么进bios的可以百度。改完之后，按F10保存重启。
## 4.给ubuntu系统分区
参考这位博主的作法分区方法是：[ubuntu18 安装注意事项](https://blog.csdn.net/qq_37258787/article/details/80270463)
- /：20GB
- 交换空间：2GB
- /boot：500MB
- /Home：余下
我这里为了快点安装好，只分了一个60GB的/根目录。然后，启动器的位置我放在了整个磁盘上...，也就是说，会覆盖掉windows的引导程序，变成grub引导启动。之后，如果删除ubuntu系统所在盘，重启电脑就会出现grub rescue的命令界面，并且进不去window10。
这里给出这个问题的解决办法：
- 第一种办法：删除ubuntu系统盘后，记得使用diskgenuis或者winPE修复C盘的MBR引导程序。（具体可参考：[删除linux系统磁盘后，开机显示grub rescue怎么办](https://blog.csdn.net/u012234115/article/details/38110613)）
- 第二种办法：给ubuntu分区的时候，分出一个/boot区域，再把启动器放在/boot上，进windows安装easybcd添加ubuntu系统启动引导。

# 二、ubuntu18 LTS安装好了，下面是一些必做的事情

## 1.更换源
 1. 备份原来的软件源
`sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak`
 2. 替换源
`sudo gedit /etc/apt/sources.list`
```oder
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
--------------------- 
作者：彼时思默 
来源：CSDN 
原文：https://blog.csdn.net/qq_37258787/article/details/80270463 
版权声明：本文为博主原创文章，转载请附上博文链接！
```
3. 更新源和软件
`sudo apt-get update`
`sudo apt-get upgrade`

## 2.设置时区为London 0时区
这是因为我们安装的是window+ubuntu双系统，在windows系统中认为bios时间是本地时间，但在ubuntu中，认为bios时间是UTC时间，当地时间为utc时间+时区数，中国（东八区）就是+8。所以，我这里将ubuntu系统时区设为0时区，这样ubuntu系统时间就和window系统显示一样了，都是bios时间。
（参考：[经验体会：解决Ubuntu 18.04+Windows双系统时间不同步的问题](https://www.colabug.com/4399564.html)）
## 3.给你的独立显卡安装专用驱动程序
>参考：[Ubuntu 18.04 NVIDIA驱动安装总结](https://blog.csdn.net/tjuyanming/article/details/80862290)

在安装之前首先就是要禁用Nouveau的驱动，禁用该驱动的方法参照[这篇博客](https://blog.csdn.net/tjuyanming/article/details/79267984)。

上一步的改动只是在安装的时候临时禁用。如果没有永久禁用该驱动，可能会出现安装完毕NIVIDA显卡后无法进入Ubuntu的情况(在登录界面，输入密码也无法登录)。

所以，在安装后Ubuntu成功后需要在grub的配置文件里面更改：

`$ sudo gedit /boot/grub/grub.cfg `

在文本中搜索quiet slash 然后添加acpi_osi=linux nomodeset，保存文本即可。

1. 使用标准Ubuntu 仓库进行自动化安装

这种方法几乎是所有的示例中最简单的方法，也是该教程最为推荐的方法。首先，检测你的NVIDIA显卡型号和推荐的驱动程序的模型。在命令行中输入如下命令：
 ```javascript
 $ ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00001180sv00001458sd0000353Cbc03sc00i00
vendor   : NVIDIA Corporation
model    : GK104 [GeForce GTX 680]
driver   : nvidia-304 - distro non-free
driver   : nvidia-340 - distro non-free
driver   : nvidia-384 - distro non-free recommended
driver   : xserver-xorg-video-nouveau - distro free builtin

== cpu-microcode.py ==
driver   : intel-microcode - distro free

```

从输出结果可以看到，目前系统已连接Nvidia GeFrand GTX 680显卡，建议安装驱动程序是 nvidia-384版本的驱动。如果您同意该建议，请再次使用Ubuntu驱动程序命令来安装所有推荐的驱动程序。
输入以下命令：

`$ sudo ubuntu-drivers autoinstall` 

一旦安装结束，重新启动系统，你就完成了。

## 4.更换grub2引导界面背景
- 复制一张图片到/boot/grub
`sudo cp pic.jpg /boot/grub`
- 更新grub2引导配置
`sudo update-grub`
- 重启你的ubuntu系统

## 5.grub2设置默认启动系统和默认等待时间
- 设置默认启动项

	`sudo gedit /etc/default/grub`

	在弹出的文件中找到GRUB_DEFAULT= 0，0代表引导菜单的第一项，我电脑上window10在第5项，所以我这里设置为4。
	
- 修改GRUB2超时时间为3秒：
	
	在grub文件中找到GRUB_TIMEOUT=10，设置为GRUB_TIMEOUT=3，表示3秒超时时间。
- 更新grub2引导配置

	`sudo update-grub`
	
- 重启完成

## 6.更换登录界面背景图片

先找到一张准备用作背景的图片，假设是picture.jpg
将该图片移动至/usr/share/backgrounds/目录下(非必须)

`sudo mv picture.jpg /usr/share/backgrounds/`

修改18.04相关配置文件：/etc/alternatives/gdm3.css

`sudo gedit /etc/alternatives/gdm3.css`

找到默认代码并修改(可以提前备份避免出错)

默认代码：

```css
#lockDialogGroup {
 background: #2c001e url(resource:///org/gnome/shell/theme/noise-texture.png);
 background-repeat: repeat; 
}
```

修改为：
```css

#lockDialogGroup {
 background: #2c001e url(file:///usr/share/backgrounds/picture.jpg);         
 background-repeat: no-repeat;
 background-size: cover;
 background-position: center; 
}
```
- 保存并重启。

## 7.gnome3更换mac os风格主题
>参考：[Ubuntu18.04完全美化教程　Ubuntu18.04主题更换为 Mac OS high Sierra](https://blog.csdn.net/dream_an/article/details/80789767)

贴图镇楼

## 8.使用deepin wine版本的安装包安装qq，微信等windows软件
安装包下载和使用说明参考：[deepin wine安装包仓库和使用教程](https://gitee.com/wszqkzqk/deepin-wine-for-ubuntu)

安装教程

    克隆 (git clone https://github.com/wszqkzqk/deepin-wine-ubuntu.git) 或下载到本地。

    在中国推荐用下面的地址，速度更快： (git clone https://gitee.com/wszqkzqk/deepin-wine-for-ubuntu.git)

    方法1(推荐)：在终端中运行（授予可执行权限后）： ./install.sh 。

    方法2：使用图形界面的软件包管理器按顺序安装所有 deb 文件。


使用说明

下载并安装所需要的deepin-wine容器 （建议在终端下使用dpkg -i安装容器，否则容易误报依赖错误）

推荐使用特别兼容支持包：

    QQ
    TIM
    QQ轻聊版
    微信
    Foxmail
    百度网盘
    360压缩
    WinRAR
    迅雷极速版

