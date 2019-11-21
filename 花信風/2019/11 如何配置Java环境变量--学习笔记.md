[pixiv: 007]: # 'https://i.loli.net/2019/06/10/5cfdc6bc7b99f37031.jpg'

# 写在前面
安装过好多次Java开发环境JDK了，但是每次都是百度如何配置环境变量的（总是记不住）。这次写一篇博客记录下配置过程，下次看自己的博客就好了-:)。
# 下载并安装JDK
在你配置Java环境变量之前，你首选要有一个JDK是吧。所以第一步是，百度Oracle Java官网，找到你要下载的JDK版本（我下的是1.8），听说1.8之后要开始收费了:sweat_smile:。
下载好之后，点击安装JDK就好了，如果你下载的不是.msi文件，而是zip的话，解压到指定目录就好了:smiley:。
# 开始配置Java环境变量
右键 我的电脑 -> 高级系统设置 -> 环境变量，就可以打开下面的环境变量配置窗口了。
![在这里插入图片描述](https://starsky1.github.io/poi/2019/071.png)
第二步，新建一个`JAVA_HOME`环境变量，值填写`D:\Java\JDK1.8`（你的填写你安装的JDK目录）如图。
![在这里插入图片描述](https://starsky1.github.io/poi/2019/072.png)
第三步，编辑path环境变量，加入`%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;`，如图：
![在这里插入图片描述](https://starsky1.github.io/poi/2019/073.png)
第四步，新建`CLASSPATH`环境变量，值填写`.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar`（注意开头有个点）：
![在这里插入图片描述](https://starsky1.github.io/poi/2019/074.png)
到现在，环境变量都配置好了，依次点击确定，关闭环境变量窗口，高级系统设置窗口。

第五步，`win键+R键`，输入`cmd`打开命令行窗口，输入`java`命令，看看有没有输出结果（效果应该像这样）。
![在这里插入图片描述](https://starsky1.github.io/poi/2019/075.png)
再输入`java -version`，看看有没有输出java的版本号：
![在这里插入图片描述](https://starsky1.github.io/poi/2019/076.png)
再输入`javac`，看看有没有输出结果，正确结果像这样：
![在这里插入图片描述](https://starsky1.github.io/poi/2019/077.png)
如果上面的三个命令都没有问题，那么恭喜你，Java环境变量配置成功:smile:，可以开始你的Java编程之路啦:kissing_heart:。
# 参考资料
> JDK安装与环境变量配置  link: https://jingyan.baidu.com/article/6dad5075d1dc40a123e36ea3.html