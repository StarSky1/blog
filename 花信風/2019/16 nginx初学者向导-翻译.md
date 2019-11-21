[pixiv: 016]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/16.png'
<div align="center"><b>初学者的向导</b></div>

这个向导给予了对nginx的基本介绍，描述了一些简单的任务，那个可以被学会在向导中。此向导假设nginx是早已被安装在阅读者的机器上。如果不是这样，请看[installing nginx](http://nginx.org/en/docs/install.html)页面。这个向导描述了如何去启动和停止nginx，重载它的配置，说明了配置文件的结构，描述了如何装配nginx去服务静态内容，如何配置nginx作为一个代理服务，以及如何连接nginx和一个FastCGI应用。

nginx有一个主要进程和一些工作者进程。主进程主要的目的是去读取和评估配置，维持工作者进程。工作者进程对客户端请求做实际的处理。nginx雇佣基于事件的模型，依赖OS的机制去有效的在工作者进程之间分发请求。工作者进程的数量被定义在配置文件中，可以被一个配置确定或者自动调节到CPU核心可用的数量(see [worker processes](http://nginx.org/en/docs/ngx_core_module.html#worker_processes))。

nginx和它的模块工作的方式被配置文件决定。默认的，配置文件被命名为nginx.conf，被放置在`/usr/local/nginx/conf`，`/etc/nginx`,或`/usr/local/etc/nginx`。

<div align="center" ><b>启动，停止，和重载配置</b></div>

要启动nginx，请运行可执行文件。一旦nginx被启动，它能控制通过调用可执行文件结合`-s`参数。使用如下的语法：
> nginx -s signal
信号的类型可能是下面其中的一个：
- stop —— 快速停止
- quit —— 优雅的停止
- reload —— 重新加载配置文件
- reopen —— 重新打开日志文件
举个例子，去停止nginx进程，等待工作进程完成正在服务的当前请求，下面的命令会被执行：
> nginx -s quit
这个命令应该被执行在相同的用户下，那个启动nginx的用户。

在配置文件上做的改变不会被应用，直到重载配置的命令被发送到nginx或者它被重启。要去重载配置，执行：
> nginx -s reload
一旦主进程接收到重载配置的命令，它会检查新配置文件的语法正确性，尝试在nginx上去应用配置。如果这是成功的，主进程会启动新的工作进程，并发送消息给旧的工作进程，请求他们去停止。否则，主进程回滚这些改变，继续使用旧的配置去工作。旧的工作进程，接收了去停止的命令，将停止接受新的连接，将继续服务当前请求，直到所有如此的请求被服务晚。在那之后，旧的工作进程将退出。

一个信号也能被发送到nginx进程，在unix工具的帮助下，例如`kill`工具。在这种情况下，一个信号被直接发送到进程通过一个进程id。nginx主进程的进程id被写，默认情况下，在nginx.pid文件，此文件在`/usr/local/nginx/logs`或`/var/run`目录中。举个例子，如果主进程id是1628，发送QUIT信号去让nginx进行一个优雅的停止，执行：
> kill -s QUIT 1628
为了得到正在运行的nginx进程列表，`ps`工具可以被使用，举个例子，像下面这种方式：
> ps -ax | grep nginx
获取更多发送信号给nginx的信息，请看[Controlling nginx](http://nginx.org/en/docs/control.html)。

<div align="center"><b>配置文件的结构</b></div>

nginx由模块组成，那个被控制通过在配置文件中的特定的指令。指令被划分为简单的指令和块指令。一个简单指令由名称和参数组成，那个被空格分隔，并以一个`;`作为结束。一个块指令和简单指令有相似的结构，但是结尾分号被替换为一个额外指令的集合，那个被环绕通过大括号(`{`和`}`)。如果一个块指令有其他的指令在括号中，它将被成为一个上下文（例子：events,http,server,and location）。

指令，那些被放置在配置文件的最外层，被认为是在主上下文环境下。`events`和`http`指令属于主作用域(上下文)，`server`在`http`中，`location`在`server`中。

在这个`#`符号之后剩下的行，被认为是注释。

<div align="center"><b>为静态内容服务</b></div>

一个重要的web server任务是部署输出文件(例如images或静态的html页面)。你将在这里实现一个例子，那个依赖于请求，文件将被从不同的本地目录发送：`/data/www`（那个可能包含html文件），`/data/images`（包含图片）。这将需要编辑配置文件，配置`server`块在`http`块中，通过使用两个`location`块。

首先，创建`/data/www`目录，并放入一个包含任意文本内容的`index.html`文件，创建`/data/images`目录，并放置一些图片。

下一步，打开配置文件。默认的配置文件早已包括了几个`server`块的例子，多半地被注释了。现在，注释掉这些块，并输入一个新的`server`块：
```
http {
	server {

	}
}
```
通常，配置文件包括一些`server`块，它们通过监听的端口和服务名称区分。一旦nginx决定那个`server`进程处理请求，它会测试这个请求头的特定uri是否和定义在location指令的参数匹配。

添加下面的`location`块，在`server`块中：
```
location / {
	root /data/www;
}
```
未完...，翻译好累。
这个`location`块指定`/`前缀，那个用于和请求路径进行比较。对于匹配的请求，uri将被添加到在根指令中指定的路径下，那就是说，在`/data/www`，形成了在本地文件系统中访问请求文件的本地路径。如果有多个匹配成功的`location`块，那么nginx会选择那个匹配结果中，前缀长度最长的一个。在上面示例的这个`location`块提供了最短的前缀，长度为1，所以只有其他的`location`块都匹配失败时，这个块才会被nginx采用。

下一步，添加第二个`location`块：
```
location /images/ {
	root /data;
}
```
它将用来匹配以`/images/`开始的请求，`location /`也能匹配这样的请求，但是`/`的前缀长度更短，nginx不会采用。

这个`server`块的配置结果应该看起来像这样：
```
server {
	location / {
		root /data/www;
	}
	location /images/ {
		root /data;
	}
}
```
这个已经是一个可以工作的server配置，那个监听在80端口，是可访问的在本地机器上通过`http://localhost`。作为uri以`/images/`开始的请求的返回结果，这个server将从`/data/images`目录发送文件。举个例子，为了响应`http://localhost/images/example.png`的请求，nginx将发送`/data/images/example.png`文件。如果这样的文件不存在，nginx将发送一个响应指示一个404错误。请求的uri不是以`/images/`开头时，将被映射到`/data/www`目录。举个例子，为了响应`http://localhost/some/example.html`的请求，nginx将发送`/data/www/some/example.html`文件。

为了应用新的配置，启动nginx如果它至今没有被启动，或者发送`reload`信号到nginx的主进程，通过执行命令：
> nginx -s reload

为了防止一些事情没有按照预期来工作，你可能想找到一些原因在`access.log`和`error.log`日志文件中，那个在`/usr/local/nginx/logs`或`/var/log/nginx`。

<div align="center"><b>搭建一个简单的代理服务器</b></div>

nginx被频繁使用的功能之一是配置它作为一个代理服务器，那个的意思是一个服务器，那个接收一些请求，传递他们的请求到被代理的服务器，从被代理服务器取回那些响应，并且发送那些响应到这些客户端。

我们将配置一个基本的代理服务器，那个服务于一些访问本地目录图片文件的请求，并将所有其他的请求发送到被代理的服务器。在这个示例中，所有的服务将被定义在单个的nginx实例上。

首先，定义被代理的服务器通过添加一个`server`块到nginx的配置文件，附带下面这些内容：
```
server {
	listen 8080;
	root /data/up1;

	location / {

	}
}
```
这个是一个简单的服务器，那个监听在8080端口（上一节，`listen`指令没有被指定，因为上一节使用的是80端口，这是默认的），并且映射所有的请求到本地文件系统的`/data/up1`目录。创建这个目录，放入一个`index.html`文件。注意，这个`root`指令被放置在`server`上下文中。这样的`root`指令会被使用，当为服务请求而选择的位置块没有包含自己的`root`指令时。

下一步，使用服务器之前部分的配置，并且修改它，添加上作为一个代理服务器的配置。在首个`location`块中，放入`proxy_pass`指令和协议号，主机名，和端口，被代理服务器被指定通过这些参数（在我的案例中，它是`http://localhost:8080`）：
```
server {
	location / {
		proxy_pass http://localhost:8080;
	}

	location /images/ {
		root /data;
	}
}
```
我们将修改第二个`location`块，那个当前映射带有`/images/`前缀的请求到`/daat/images`目录下的文件，为了使`location`块匹配那些images请求中特定的文件扩展名。对`location`块做出看起来像这样的修改：
```
location ~\.(gif|jpg|png)$ {
	root /data/images;
}
```
这个参数是一个正则表达式，那个匹配所有那些以`.gif`，`.jpg`或`.png`结束的请求uri。正则表达式的前面应该放置`~`。这些相应的请求将被映射到`/data/images`目录。

当nginx选择了一个`location`块去服务一个请求时，它首选会检查带有详细前缀的`location`指令，并记住那个具有最长前缀的`location`，然后检查正则表达式。如果有一个匹配的正则表达式，nginx将选择这个`location`，否则，它会选择之前记住的那个`location`。

这个代理服务器的配置结果看起来像这样：
```
server {
	location / {
		proxy_pass http://localhost:8080/;
	}

	location ~\.(gif|jpg|png)$ {
		root /data/images;
	}
}
```
这个服务将过滤那些以`.gif`，`.jpg`或`.png`结束的请求，并映射他们到`/data/images`目录（通过添加uri到`root`指令的参数）并且传递其他的请求到上面配置的被代理服务器中。

为了应用新的配置，发送`reload`信号给nginx，像之前的小节描述的那样。

还有[更多](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)的指令，那些可以被用来去更深层次的配置一个代理连接。

<div align="center"><b>搭建一个FastCGI 代理</b></div>

nginx可以被使用去路由一些请求到FastCGI服务器，那个运行一些使用多样的框架和编程语言构建的应用，例如PHP。

这个最基础的nginx配置，那个用于搭建一个FastCGI服务，包括使用`fastcgi_pass`指令代替`proxy_pass`指令，还有`fastcgi_param`指令，那个设置一些参数传递到一个FastCGI服务上。假设这个FastCGI服务是可访问的，路径是`localhost:9000`。采用之前章节的代理配置作为一个基础，替换`proxy_pass`指令为`fastcgi_pass`指令，并且改变参数为`localhost:9000`。在PHP中，这个`SCRIPT_FILENAME`参数被用来决定脚本的名字，`QUERY_STRING`参数被用来传递请求参数。配置结果会像这样：
```
server {
    location / {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param QUERY_STRING    $query_string;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```
这个将搭建一个服务，那个将通过FastCGI协议将除静态图像请求外的所有请求路由到在localhost:9000上运行的被代理的服务器。


# 原文章 nginx官方文档
- [Beginner’s Guide](http://nginx.org/en/docs/beginners_guide.html)

