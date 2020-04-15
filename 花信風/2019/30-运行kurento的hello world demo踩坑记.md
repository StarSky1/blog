[pixiv: 030]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/30.jpg'

@[TOC](踩坑记)

# 前言

说说遇到的困难。第一个，kurento 官网提供了丰富的文档，例如入门引导、安装引导、使用示例等等，可以说很友好了。但是，官方文档都是英文的，对于我这种英语不好的，阅读起来就很吃力了。（最终使用 google 翻译和自己的一点词汇量，使用好几个小时才艰难的看懂了一些）。第二个，要使用kms(kurento media server) 流媒体服务器，需要在 ubuntu 上安装 kms、turn 服务器（我是在阿里云服务器上安装的），自己电脑要运行 helloworld 示例代码。安装配置步骤过多，再加上第一次接触 kurento，导致中间遇到许多问题，花费了很多时间。

## 安装 kms 

按照 [kms安装向导](https://doc-kurento.readthedocs.io/en/6.13.0/user/installation.html)，我们有三种方式进行 kms 的安装。
1. 在 Amazon Web Services (AWS) 上安装 
2. docker安装
3. 本地安装，支持 Ubuntu 16.04 (Xenial) and Ubuntu 18.04 (Bionic) (64-bits only)

我做测试，图方便就用 docker 安装了。另外说一下，我用的是阿里云服务器安装的，服务器系统是 ubuntu16.04。
```
# docker 安装 kms 命令
docker run --name kms -d -p 8888:8888     kurento/kurento-media-server
```
按照 [STUN/TURN服务安装（for Kurento Media Server）](http://www.voidcn.com/article/p-kicyxdel-bow.html) 的指导，安装 turn 服务器。
> 如果您在NAT环境（即任何云提供商）中安装Kurento，您需要在/etc/kurento/modules/kurento/WebRtcEndpoint.conf.ini中提供STUN服务器配置。 除此之外，您将必须打开您的安全组中的所有UDP端口，因为STUN将使用从整个0-65535范围可用的任何端口。
>  下列端口应该在防火墙中打开：
>- 3478 TCP & UDP  
>- 49152 - 65535 UDP: 根据RFC 5766，这些是TURN服务器将用于交换媒体的端口。 可以使用turnserver中的--max-port和--min-port选项更改这些端口。
>虽然RFC指定TURN使用的端口，但如果使用STUN，则需要打开所有UDP端口，因为这些端口不受约束。

安装 turn 服务器
```
apt update
apt install coturn

 安装软件包后，您需要修改/etc/init.d/coturn中的启动脚本。
 
将外部和本地IP添加到启动脚本中：
EXTERNAL_IP=外网地址
LOCAL_IP=内网地址

启动脚本中修改 DAEMON_ARGS 变量，通过 DAEMON_ARGS 变量来修改考虑在内的 IPs ，以及长期用户和密码凭据（在此情况下为 kurento:kurento ，但可能不同），realm 和其他一些选项：
DAEMON_ARGS="-c /etc/turnserver.conf -f -o -a -v -r kurento.org -u kurento:kurento --no-stdout-log --external-ip $EXTERNAL_IP/$LOCAL_IP"

然后让我们启用turnserver作为自动服务守护程序运行。 为此，打开文件/etc/default/coturn并取消注释键：
TURNSERVER_ENABLED=1

复制DTLS、TLS支持的证书文件：
cp /usr/share/coturn/examples/etc/turn_server_cert.pem /etc/turn_server_cert.pem
cp /usr/share/coturn/examples/etc/turn_server_pkey.pem /etc/turn_server_pkey.pem

现在，你要告诉 KMS 服务器 TURN 已经安装。为此，修改 /etc/kurento/modules/kurento/ WebRtcEndpoint.conf.ini文件中的 turnurl 键值：
turnURL=kurento:kurento@<public-ip>:3478
stunServerAddress=<public-ip>
stunServerPort=3478

修改docker运行的kms配置文件方法
1.复制容器中的 配置文件到 宿主机中
docker cp  kms:/etc/kurento/modules/kurento/WebRtcEndpoint.conf.ini  ~/WebRtcEndpoint.conf.ini
2.修改配置文件
vim ~/WebRtcEndpoint.conf.ini
3.宿主机配置文件 复制到 kms容器中
 docker cp ~/WebRtcEndpoint.conf.ini kms:/etc/kurento/modules/kurento/WebRtcEndpoint.conf.ini
 4.重启 kms 容器
 docker restart kms

最后要做的是启动 coturn 服务器和媒体服务器：
sudo service coturn start 

#我们使用 docker 启动了 kms 容器，不是本地安装， 下面这一步就不需要了
sudo service kurento-media-server-6.0 restart
```
使用[此测试应用程序](http://www.voidcn.com/link?url=https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/)检查您的安装


## 运行 hello world 示例

按照 [Kurento Java Tutorial - Hello World](https://doc-kurento.readthedocs.io/en/6.13.0/tutorials/java/tutorial-helloworld.html)文档，运行 hello world 示例。
1. 安装 Kurento Media Server: [安装向导](https://doc-kurento.readthedocs.io/en/6.13.0/user/installation.html).
2. git bash 中运行这些命令:
```
git clone https://github.com/Kurento/kurento-tutorial-java.git
cd kurento-tutorial-java/kurento-hello-world
git checkout 6.13.0
# 这一步不需要，我们用idea来运行 mvn -U clean spring-boot:run -Dkms.url=ws://localhost:8888/kurento

用 idea 打开 kurento-hello-world 项目，下载 maven 依赖完成后，在 run configurations 中添加 new configurations。
选择 maven , command line 中输入：
spring-boot:run -Dkms.url=ws://<public-ip>:8888/kurento  # 这里的 public-ip 添你安装 kms 的服务器公网 ip 地址

打开 浏览器，访问 https://localhost:8443/
点击 Start 按钮，就可以 查看 demo 效果了。
```
运行 hello world 示例时，会遇到 使用websocket传输内容过长的问题
解决方法，参考 [springboot 框架中使用 websocket 传输内容过长的问题解决](https://blog.csdn.net/github_39083395/article/details/103271398)
```java
//项目中创建 package org.kurento.tutorial.helloworld.config
//config 包下创建类
package org.kurento.tutorial.helloworld.config;

/**
 * @Auther: starsky1
 * @Date: 2020/3/27/0027 19:03
 * @Description: todo
 */
@Configuration
//@ComponentScan
//@EnableAutoConfiguration
public class WebAppRootContext implements ServletContextInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        System.out.println("org.apache.tomcat.websocket.textBufferSize");
        servletContext.addListener(WebAppRootListener.class);
        servletContext.setInitParameter("org.apache.tomcat.websocket.textBufferSize","1024000");
    }

}
//此问题即可解决
```
![run configurations配置](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/30/3001.png)
![demo 运行效果](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/30/3002.png)
