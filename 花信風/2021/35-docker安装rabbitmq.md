[pixiv: 035]: # "https://cdn.jsdelivr.net/gh/starsky1/poi/2021/35.jpg"
 
1、创建需要挂载的rabbitmq目录
```bash
mkdir -p ~/rabbitmq/{etc,lib,log}
```
2、给创建的文件夹授权任何人，读写可执行权限
```bash
chmod -R 777 ~/rabbitmq 
```

防火墙没启动？用命令启动： `systemctl start firewalld` 

3、开放 15672、5672端口，然后重新加载防火墙配置 (15672是rabbitmq后台管理界面访问端口,5672是应用端访问rabbitmq服务端口)
```bash
firewall-cmd --zone=public --add-port=15672/tcp --permanent

firewall-cmd --zone=public --add-port=5672/tcp --permanent

firewall-cmd --reload
```

4、启动 rabbitmq容器，注意使用带management标签的镜像，这样才有后台管理界面
```bash
docker run -d --name rabbitmq --restart=always -p 5672:5672 -p 15672:15672 \
-v ~/rabbitmq/etc:/etc/rabbitmq -v ~/rabbitmq/lib:/var/lib/rabbitmq -v ~/rabbitmq/log:/var/log/rabbitmq \
--hostname myRabbit -e RABBITMQ_DEFAULT_VHOST=my_vhost  -e RABBITMQ_DEFAULT_USER=admin \
 -e RABBITMQ_DEFAULT_PASS=123456 rabbitmq:management
```
5、启动 rabbitmq后台管理页面插件
```bash
docker exec -it rabbitmq rabbitmq-plugins enable rabbitmq_management
```

启动centos7防火墙
`systemctl start firewalld` 



### 解决报错WARNING: IPv4 forwarding is disabled. Networking will not work.
[docker容器启动报错解决方法：](https://www.cnblogs.com/python-wen/p/11224828.html)