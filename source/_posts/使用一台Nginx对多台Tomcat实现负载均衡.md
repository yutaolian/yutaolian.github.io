---
title: 使用一台Nginx对多台Tomcat实现负载均衡
date: 2016-10-28 14:00:38
tags:
	- Nginx
	- 负载均衡
	- Tomcat
---

### 0. 简介

使用一台Nginx服务器通过配置多台tomcat实现访问负载均衡。其实网上有很多例子。之前也配置过。不过为了加强记忆。写篇文章记录要点。(实现环境为mac pro)

### 1.安装tomcat(本例安装2台)
直接去[Apache tomcat](http://tomcat.apache.org/download-70.cgi)下载zip包，![](/img/nginx/nginx_01.png)
找个文件夹解压并且复制一份（可复制多个）如图
![](/img/nginx/nginx_02.png)
### 2.修改tomcat的端口号
对于搞Java web的小伙伴这就不详细说了。
如图所示修改server.xml文件
![](/img/nginx/nginx_03.png)
端口号可随意修改（建议改的大一点）。只要不冲突就OK
看过server.xml文件的小伙伴都知道里面有三个端口号:
	
	8005，8080，8009

这三个端口号分别代表什么意思暂且不说。
第一台tomcat不做修改。第二台改成如图所示。当然可以改成你想要的端口号

![](/img/nginx/nginx_04.png)

![](/img/nginx/nginx_05.png)

![](/img/nginx/nginx_06.png)

### 3.启动tomcat
mac中zip包解压后默认bin中的命令是不可以执行的。如图
![](/img/nginx/nginx_07.png)
修改bin中的命令的执行权限然后启动tomcat
（使用该命令必须在当前tomcat的bin目录下）
```
	chmod 777 *.sh
```
![](/img/nginx/nginx_08.png)

分别在tomcat的bin目录下执行

```
	sh startup.sh
	//sh shutdown.sh --停止tomcat
```

在两个tomcat的webapps目录下新建一个文件夹名为test.在test中新建文件test.html
作为测试tomcat启动后的测试页面。

![](/img/nginx/nginx_12.png)

	分别在body中输入<h2>this is tomcat 1</h2> 和 <h2>this is tomcat 2</h2>

如图所示
![](/img/nginx/nginx_13.png)

在浏览器中分别输入 ：
http://localhost:8080/test/test.html
http://localhost:8081/test/test.html

出现如下图时。tomcat安装成功。

![](/img/nginx/nginx_09.png)
![](/img/nginx/nginx_10.png)


### 4.安装nginx

在mac上安装nginx非常简单 使用brew安装即可

```
brew install nginx
```
OK nginx 安装成功
### 5.启动nginx
启动命令：

```
	nginx
```
停止命名：

```
 nginx -s stop
```
默认nginx监听80端口。
启动后在浏览器输入：http://localhost
出现如图择nginx 启动成功
![](/img/nginx/nginx_11.png)

### 6.配置负载（也是本blog最重要的点）

找到nginx的配置文件:nginx.conf
brew安装后默认的配置文件路径为：/usr/local/etc/nginx
修改nginx.conf文件如图所示

![](/img/nginx/nginx_14.png)

```
	 upstream myTomcat {
   	 	server 127.0.0.1:8080 weight=5;
   	 	server 127.0.0.1:8081 weight=5;
    }
    server {
        listen       8888;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            #root   html;
            #index  index.html index.htm;
       	proxy_pass http://myTomcat;

        }
        ...其余代码省略
```

### 7.验证负载

在浏览器中输入 [http://localhost:8888/test/test.html](http://localhost:8888/test/test.html)
可打开多个tab也测试。会发现tomcat1 和tomcat2在切换。

![](/img/nginx/nginx_15.png)

OK。这样就实现了简单的负载均衡。

PS: nginx的配置需继续研究。






