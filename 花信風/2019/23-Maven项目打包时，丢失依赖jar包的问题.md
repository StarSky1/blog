[pixiv: 023]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/23.jpeg'
# 问题
本来准备给Maven项目打个jar包，然后上传到linux服务器运行。结果发现，mvn package打好的jar包只有自己写的源代码，依赖的那些jar包都丢失了。导致我运行jar包时，报`classNotFoundException：io.netty`。刚开始还以为是代码出问题了，经过一番分析才发现，原来我打的jar包里只有自己写的代码。。。
![难受](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2301.jpg)
# 解决方案
通过一番搜索，终于找到了一个可行的解决方案。那就是通过maven的`maven-assembly-plugin`插件，注意版本号是`2.5.5`，我测试过`3.5.0`竟然不行，`3.5`打包到一半会卡住。
### 一、添加maven插件
下面是pom.xml文件的`build`块代码：
```java
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.5.5</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.yj.TCPClient.upload.App</mainClass>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <!-- 添加此项后，可直接使用mvn package | mvn install -->
                <!-- 不添加此项，需直接使用mvn package assembly:single -->
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>

    </build>
```
### 二、使用mvn package打包运行
我这里使用idea的maven操作台来打包：
![maven操作台](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2302.png)
`注意：点击右上角的闪电⚡符号可以跳过测试哦`

### 三、打包完毕，复制jar包到linux服务器运行
打包成功之后（build success），就可以在项目的target目录下看到我们打好的jar包了，名字应该叫`TCP_Client-1.0.0-jar-with-dependencies.jar`。然后，我复制这个jar包到linux服务器运行就好了，所有依赖项都在里面了。

# 总结
Maven这个东西真玄学，为了给maven项目打包，我都折腾了三小时了，真难受啊。
![困](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2303.gif)
