# Docker Java教程5:使用Docker Compose 实现多容器应用

tags ： [docker]

---
<h2 id="1.1"> 什么是Docker Compose？</h2>

Docker Compose 是一个使用Docker来定义和运行复杂应用的工具.
通过Compose,你能在一个单独的文件中定义一个多容器应用,通过一个命令就能完成所有事情，并使它们运行起来.

— github.com/docker/compose

<!--more-->

[什么是Docker Compose](#1.1)

[配置文件](#1.2)

[启动应用](#1.3)

[验证应用](#1.4)

[停止应用](#1.5)

一个使用Docker容器技术的应用一般都会包含多个容器.通过Docker Compose,没有必要去写shell 脚本来启动你的容器.所有的容器都通过services定义在一个配置文件当中,然后docker-compose 脚本就可以用来启动，停止，或者重启应用，这样一来，所有的容器就集成在service当中，而所有的services则被包含在应用当中.
完整的命令列表如下所示:

|Command|Purpose
|:----:|:----:
|build|Build or rebuild services
|help|Get help on a command
|kill|Kill containers
|logs|View output from containers
|port|Print the public port for a port binding
|ps|List containers
|pull|Pulls service images
|restart|Restart services
|rm|Remove stopped containers
|run|Run a one-off command
|scale|Set number of containers for a service
|start|Start services
|stop|Stop services
|up|Create and start containers

这个章节使用的应用是一个与数据库交互的Java EE应用.
这个应用发布一个REST节点，可以通过`curl`调用.它通过[WildFly Swarm](http://wildfly-swarm.io/)部署，并且与MySQL数据库进行交互.

WildFly Swarm 和 MySQL 将会在两个隔离的容器当中运行,因此这是一个多容器应用.

<h2 id="1.2">配置文件</h2>

Docker Compose 的入口是一个Compose 文件,通常命名为`docker-compose.yml`.创建一个新目录`javaee`,在该目录创建一个`docker-compose.yml`文件，内容如下:

    version: '3.3'
    services:
      db:
        container_name: db
        image: mysql:8
        environment:
          MYSQL_DATABASE: employees
          MYSQL_USER: mysql
          MYSQL_PASSWORD: mysql
          MYSQL_ROOT_PASSWORD: supersecret
        ports:
          - 3306:3306
      web:
        image: arungupta/docker-javaee:dockerconeu17
        ports:
          - 8080:8080
          - 9990:9990
        depends_on:
          - db

在这个Compose文件当中:

1. 在这个Compose 文件当中定义了两个services，分别是 db 和 web.

2. 每个service的镜像名称通过image 属性进行定义.
MYSQL_ROOT_PASSWORD is mandatory and specifies the password that will be set for the MySQL root superuser account.

3. mysql:8 镜像 启动 MySQL 服务器.

4. 环境变量属性定义环境变量来初始化MySQL 服务器.
   * MYSQL_DATABASE 允许你在镜像启动的时候指定被创建的数据库的名称.
   * MYSQL_USER 和 MYSQL_PASSWORD 用来创建一个新用户并且赋予那个用户的密码.这个用户将会赋予MYSQL_DATABASE属性指定的数据库的超级管理员权限.
   * MYSQL_ROOT_PASSWORD 是强制必填项,并且将会设置为MySQL root 超级用户账号的密码.

5. Java EE 应用 使用的 `db` 服务通过 `connection-url` 进行指定https://github.com/arun-gupta/docker-javaee/blob/master/employees/src/main/resources/project-defaults.yml/.

6. arungupta/docker-javaee:dockerconeu17 image 启动 WildFly Swarm 应用服务器. 它包含从 https://github.com/arun-gupta/docker-javaee 构建的Java EE 应用. 如果你想要构建自己的镜像可以Clone这个项目.

7. 端口跳转通过`ports`属性来完成.

8. `depends_on` 属性允许表达services之间的依赖关系.在这个例子中,MySQL将会在WildFly之前启动. 

<h2 id="1.3">启动应用</h2>

在这个应用当中的所有services都能在detached模式被启动，通过以下命令:

    docker-compose up -d

一个可替代的Compose文件名称能通过`-f`标志来指定.

一个可选的compose文件存在的目录能够通过`-p`标志来指定.

输出如下:

    docker-compose up -d
    Creating network "javaee_default" with the default driver
    Creating db ...
    Creating db ... done
    Creating javaee_web_1 ...
    Creating javaee_web_1 ... done
     

如果镜像已经被下载输出会稍有不同.

使用`docker-compose ps`命令验证已启动的服务.

    Name                  Command               State                       Ports
	------------------------------------------------------------------------------------------------------
	db             docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp
	javaee_web_1   /bin/sh -c java -jar /opt/ ...   Up      0.0.0.0:8080->8080/tcp, 0.0.0.0:9990->9990/tcp

还有另外一种方式,在这个应用的所有容器,和其它所有运行在这个Docker host 的容器都能通过使用`docker container ls`命令来验证:

	 Name                  Command               State                       Ports
	------------------------------------------------------------------------------------------------------
	db             docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp
	javaee_web_1   /bin/sh -c java -jar /opt/ ...   Up      0.0.0.0:8080->8080/tcp, 0.0.0.0:9990->9990/tcp
	javaee $ docker container ls
	CONTAINER ID        IMAGE                                   COMMAND                  CREATED             STATUS              PORTS                                            NAMES
	e862a5eb9484        arungupta/docker-javaee:dockerconeu17   "/bin/sh -c 'java ..."   38 seconds ago      Up 36 seconds       0.0.0.0:8080->8080/tcp, 0.0.0.0:9990->9990/tcp   javaee_web_1
	08792c20c066        mysql:8                                 "docker-entrypoint..."   39 seconds ago      Up 37 seconds       0.0.0.0:3306->3306/tcp                           db

使用`docker-compose logs`命令能够查看所有Service 的日志:

	Attaching to dockerjavaee_web_1, db
	web_1  | 23:54:21,584 INFO  [org.jboss.msc] (main) JBoss MSC version 1.2.6.Final
	web_1  | 23:54:21,688 INFO  [org.jboss.as] (MSC service thread 1-8) WFLYSRV0049: WildFly Core 2.0.10.Final "Kenny" starting
	web_1  | 2017-10-06 23:54:22,687 INFO  [org.wildfly.extension.io] (ServerService Thread Pool -- 20) WFLYIO001: Worker 'default' has auto-configured to 8 core threads with 64 task threads based on your 4 available processors

`depends_on` 属性在Compose 定义文件中是用来确保容器的启动顺序.但是应用等级的启动顺序还是需要依赖于这些应用本身在容器当中的启动顺序.在我们这个例子中WildFly Swarm 通过使用定义在https://reference.wildfly-swarm.io/fractions/datasources.html的`swarm.datasources.data-sources.KEY.stale-connection-checker-class-name` 来完成.


<h2 id="1.4">验证应用</h2>

现在WildFly Swarm 和 MySQL 已经被配置好了.让我们访问这个应用.你需要指定host主机 上WildFlay 运行的IP 地址(在我们的例子中是`localhost`).


访问地址如下:

	curl -v http://localhost:8080/resources/employees

输出如下:

	curl -v http://localhost:8080/resources/employees
	*   Trying ::1...
	* TCP_NODELAY set
	* Connected to localhost (::1) port 8080 (#0)
	> GET /resources/employees HTTP/1.1
	> Host: localhost:8080
	> User-Agent: curl/7.51.0
	> Accept: */*
	>
	< HTTP/1.1 200 OK
	< Connection: keep-alive
	< Content-Type: application/xml
	< Content-Length: 478
	< Date: Sat, 07 Oct 2017 00:05:41 GMT
	<
	* Curl_http_done: called premature == 0
	* Connection #0 to host localhost left intact
	<?xml version="1.0" encoding="UTF-8" standalone="yes"?><collection><employee><id>1</id><name>Penny</name></employee><employee><id>2</id><name>Sheldon</name></employee><employee><id>3</id><name>Amy</name></employee><employee><id>4</id><name>Leonard</name></employee><employee><id>5</id><name>Bernadette</name></employee><employee><id>6</id><name>Raj</name></employee><employee><id>7</id><name>Howard</name></employee><employee><id>8</id><name>Priya</name></employee></collection>

这里展示了查询数据库的结果:

可以获取一个单独的资源:

	curl -v http://localhost:8080/resources/employees/1

输出如下:

	curl -v http://localhost:8080/resources/employees/1
	*   Trying ::1...
	* TCP_NODELAY set
	* Connected to localhost (::1) port 8080 (#0)
	> GET /resources/employees/1 HTTP/1.1
	> Host: localhost:8080
	> User-Agent: curl/7.51.0
	> Accept: */*
	>
	< HTTP/1.1 200 OK
	< Connection: keep-alive
	< Content-Type: application/xml
	< Content-Length: 104
	< Date: Sat, 07 Oct 2017 00:06:33 GMT
	<
	* Curl_http_done: called premature == 0
	* Connection #0 to host localhost left intact
	<?xml version="1.0" encoding="UTF-8" standalone="yes"?><employee><id>1</id><name>Penny</name></employee>

<h2 id="1.5">停止应用</h2>

使用`docker-compose down`命令关闭应用:

	Stopping javaee_web_1 ... done
	Stopping db           ... done
	Removing javaee_web_1 ... done
	Removing db           ... done
	Removing network javaee_default

这会停止在每个service中的所有容器并且移除所有的services.它同样会删除所有已经被创建的作为应用一部分的网络通信.