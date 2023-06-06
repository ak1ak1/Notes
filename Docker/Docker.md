# 一、安装Docker

```shell
# 1.yum包更新到最新
yum update
# 2.安装需要的软件包，yum-util提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
yum install -y yum-utils device-mapper-persistent-data lvm2
# 3.设置yum源
yum-config-mapper --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 4.安装docker，一直y
yum install -y docker-ce
# 5.查看docker版本，验证是否成功
docker -v
```

# 二、Docker架构



![image-20230517185342362](G:\笔记\Docker\assets\image-20230517185342362.png)



- 镜像（Image）：Docker镜像，就相当于是一个root文件系统。比如官方镜像ubuntu：16.04就包含了完整的一套Ubuntu16.04最小系统的root文件系统。
- 容器（Container）：镜像和容器的关系，就像是类和对象一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、删除、暂停等。
- 仓库（Registry）：仓库可以看成是一个代码控制中心，保存镜像。

# 三、Docker命令

## 1.Docker服务命令

- 启动docker服务：systemctl start docker
- 停止docker服务：systemctl stop docker
- 重启docker服务：systemctl restart docker
- 查看docker服务状态：systemctl status docker
- 开机启动docker服务：systemctl enable docker

## 2.Docker镜像命令

- 查看镜像：docker images
- 搜索镜像：docker search 镜像名称
- 拉取镜像：docker pull 镜像名称[:版本号]    （如果没有版本号，默认是最新）
- 删除镜像：docker rmi 镜像id   或者 docker rmi 镜像名称:版本号

## 3.Docker容器命令

- 查看容器：

```shell
docker ps # 查看正在运行的容器
docker ps -a # 查看所有容器
```

- 创建容器：

```shell
docker run 参数
# 参数说明
	-i ：保持容器运行。通常与-t同时使用。加入-it这两个参数后，容器创建后自动进入容器，退出容器后容器自动关闭。
	-t ：为容器重新分配一个伪输入终端，通常与-i同时使用。
	-d ：以守护（后台）模式运行容器，创建一个容器在后台运行，需要使用docker exec进入容器。退出后，容器不会关闭。
	-it：创建的容器一般称为交互式容器
	-id：创建的容器一般成为守护式容器
	--name：为创建的容器命名
# eg
sudo docker run -id --name=test redis:latest /bin/bash
```

- 进入容器：

```shell
docker exec 参数 容器名称
```

- 启动容器：

```shell
docker start 容器名称
```

- 停止容器：

```shell
docker stop 容器名称
```

- 删除容器：

```shell
docker rm 容器名称
```

- 查看容器信息：

```shell
docker inspect 容器名称
```

# 四、Docker容器的数据卷

## 1.数据卷的概念以及作用

### 1.1问题

- Docker容器删除后，在容器中产生的数据也随之销毁
- Docker容器和外部机器可以直接交互文件嘛？
- 容器之间想要进行数据交互？

![image-20230518121412628](G:\笔记\Docker\assets\image-20230518121412628.png)

### 1.2数据卷

- 数据卷是宿主机中的一个目录或文件
- 当容器目录和数据卷目录绑定后，对方的修改会立即同步

### 1.3配置数据卷

- 创建启动容器时，使用-V参数设置数据卷

```shell
docker rum ... -v 宿主根目录(文件) :容器内目录(文件) ...
```

- 注意事项：
  - 目录必须是绝对路径
  - 如果目录不存在，会自动创建
  - 可以挂载多个数据卷

- 案例：

```shell
# 单个容器挂载单个数据卷
docker run -it --name=test -v /root/data:/root/data centos:7
# 单个容器挂载多个数据卷
docker run -it --name=test \
-v /root/data1:/root/data1 \
-v /root/data2:/root/data2 \
centos:7 
# 多个容器数据交换
docker run -it --name=test1 -v /root/data:/root/data centos:7
docker run -it --name=test2 -v /root/data:/root/data centos:7
```

## 2.数据卷容器

多容器进行数据交换：

- 多个容器挂载同一个数据卷（麻烦）
- 数据卷容器

配置数据卷容器：

1. 创建启动c3数据卷容器，使用 -v 参数设置数据卷（/volume是容器内的目录，同时会在宿主机中新增一个目录）

```shell
docker run -it --name=c3 -v /volume centos:8 /bin/bash
```

2. 创建启动c1c2容器，使用--volumes-from参数设置数据卷

```shell
docker run -it --name=c1 --volume-from c3 centos:7 /bin/bash
docker run -it --name=c2 --volume-from c3 centos:7 /bin/bash
```

## 3.小结

数据卷概念：

- 宿主机的一个目录或文件

数据卷作用：

- 容器数据持久化
- 客户端和容器数据交换
- 容器间数据交换

数据卷容器：

- 创建一个容器，挂载一个目录，让其他容器继承自该容器（--volume-from）
- 通过简单方式实现数据卷配置

# 五、应用部署

## 1.MySQL部署

![image-20230518165541863](G:\笔记\Docker\assets\image-20230518165541863.png)

- 容器内的网络服务和外部机器不能直接通信
- 外部机器和宿主机可以直接通信
- 宿主机和容器可以直接通信
- 当容器中的网络服务需要被外部机器访问时，可以将容器中提供服务的端口映射到宿主机的端口上。外部机器访问宿主机的该端口，从而间接访问容器的服务。
- 称为**端口映射**

部署Mysql步骤：

- 搜索mysql镜像

```shell
docker search mysql
```

- 拉取mysql镜像

```shell
docker pull mysql:5.6
```

- 创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建mysql目录用于存储mysql数据信息
mkdir ~/mysql_test
cd ~/mysql_test
```

```shell
docker run -id \
-p 3307:3306 \
--name=mysql_test \
-v $PWD/conf:/etc/mysql/conf.d \
-v $PWD/logs:/logs \
-v $PWD/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=fengweim \
mysql:5.6
```

- 参数说明：
  - **-p 3307:3306**：将容器的3306端口映射到宿主机的3307端口
  - **-v $PWD/conf:/etc/mysql/conf.d**：将主机当前目录下（/root/mysql_test）下的/conf/my.cnf挂载到容器的/etc/mysql/
  - **-v $PWD/logs:/logs**：将主机当前目录下的logs目录挂载到容器的/logs。日志目录。
  - **-v $PWD/data:/var/lib/mysql**：将主机当前目录下的data目录挂载到容器的/var/lib/mysql。
  - **-e MYSQL_ROOT_PASSWORD = fengweim**：初始化用户密码

## 2.Tomcat部署

- 搜索tomcat镜像

```shell
docker search tomcat
```

- 拉取tomcat镜像

```shell
docker pull tomcat:latest
```

- 创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建tomcat目录用于存储tomcat数据信息
mkdir ~/tomcat/tomcat_test
cd ~/tomcat/tomcat_test
```

```shell
docker run -id --name=tomcat_test \
-p 8080:8080 \
-v $PWD:/user/local/tomcat/webappss \
tomcat
```

### 2.1 踩坑

坑：

​	部署tomcat容器后，连接不上端口。

原因：

​	由于云服务器的检查措施一般都是两层，第一层就是防火墙，第二层是安全组。

方案：

- 配置防火墙，开通端口

  - 检查防火墙状态是否开启：

  ```shell
  systemctl status firewalld
  # 如果出现unit firewalld.service could not be found.
  # 说明没有安装防火墙
  # 下载防火墙
  apt-get install firewalld firewall-config
  ```

  - 启动防火墙

  ```shell
  service firewalld start
  ```

  - 将tomcat部署到的8080端口打开

  ```shell
  firewall-cmd --zone=public --permanent --add-port=8080/tcp
  # 注意，命令开头是firewall，不再是firewalld
  ```

  - 端口开放后将该效果重新刷新

  ```shell
  firewall-cmd --reload
  ```

  - 重新刷新后可以查看已经开通的端口

  ```shell
  firewall-cmd --list-all
  # 可以找到 public.port: 8080
  ```

- 配置安全组

- 重启docker服务

  - 重启docker服务的原因：

    - docker端口映射或启动容器时报错：driver failed programming external connectivity on endpoint quirky_allen

    - docker服务启动时定义的自定义链DOCKER由于centos7 firewall被清掉
    - firewall的底层是使用iptables进行数据过滤的，建立在iptables之，这可能会与Docker起冲突
    - 当firewalld启动或重启时，将会从iptables中移除DOCKER的规则，从而影响Docker的正常工作
    - 当你使用的是Systemd的时候，firewalld会在Docker之前启动，但是如果你在Docker启动之后在启动或者重启firewalld，你就需要重启Docker进程。
    - 重启Docker服务可重新生成自定义链DOCKER

## 3.Nginx部署

- 搜索nginx镜像

```shell
docker search nginx
```

- 拉取nginx镜像

```shell
docker pull nginx
```

- 创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建nginx目录用于存储nginx数据信息
mkdir ~/nginx/nginx_test
cd ~/nginx/nginx_test
mkdir conf
cd conf
# 在~/nginx/conf/下创建nginx.conf文件，粘贴以下内容
vim nginx.conf
```

```shell
nginx配置文件{待补充}
```

```shell
docker run --id --name=nginx_test \
-p 80:80 \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/logs:/var/log/nginx \
-v $PWD/html:/user/share/nginxhtml \
nginx
```

## 4.Redis部署

- 搜索redis镜像

```shell
docker search redis
```

- 拉取redis镜像

```shell
docker pull redis:5.0
```

- 创建容器，设置端口映射

```shell
docker run -id --name=redis_test -p 6379:6379 redis:5.0
```

- 使用外部机器连接redis

```shell
./redis-cli.exe -h xxx.xxx.xxx.xxx -p 6379
```

# 六、DockerfIle

## 1.Docker镜像原理

Linux文件系统由bootfs和rootfs两部分组成：

- bootfs：包含bootloader（引导加载程序）和kernel（内核）
- rootfs：root文件系统，包含的就是典型Linux系统中的/dev，/proc，/bin，/etc等标准目录和文件
- 不同的Linux发行版，bootfs基本一样，而rootfs不同，如ubuntu、centos等

Docker镜像原理：（复用！）

- Docker镜像是由特殊的文件系统叠加而成的
- 最底端是bootfs，并使用宿主机的bootfs
- 第二层是root文件系统rootfs，称为base image
- 然后再往上可以叠加其他的镜像文件
- 统一文件系统（union file system）技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统。
- 一个镜像可以放在另一个镜像上。位于下面的镜像称为父镜像，最底层的镜像称为基础镜像。
- 当从一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器

## 2.镜像制作

### 2.1容器->镜像->压缩文件->镜像

容器转为镜像：

```shell
docker commit 容器id 镜像名称:版本号
```

镜像转为压缩文件：

```shell
docker save -o 压缩文件名称 镜像名称:版本号
```

压缩文件转为镜像：

```shell
docker load -i 压缩文件名称
```

### 2.2Dockerfile概念

概念：

- Dockerfile是一个文本文件
- 包含一条条指令
- 每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像
- 对于开发人员：可以为开发团队提供一个完全一致的开发环境
- 对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作
- 对于运维人员：在部署时，可以实现应用的无缝移植

关键字：

| 关键字      | 作用                     | 备注                                                         |
| ----------- | ------------------------ | ------------------------------------------------------------ |
| FROM        | 指定父镜像               | 指定dockerfile基于哪个image构建                              |
| MAINTAINER  | 作者信息                 | 标明这个dockerfile是谁写的                                   |
| LABEL       | 标签                     | 用来标明dockerfile的标签，可以使用Label代替Maintainer 最终都可以在docker image基本信息中查看 |
| RUN         | 执行命令                 | 执行一段命令，默认是/bin/bash。格式：RUN command 或者 RUN[“command”，“param1”，“param2”] |
| CMD         | 容器启动命令             | 提供启动容器时的默认命令，和ENTRYPOINT配合使用。格式： CMD command param1 param2 或者 CMD ["command","param1","param2"] |
| ENTRYPOINT  | 入口                     | 一般在制作一些执行就关闭的容器中使用                         |
| COPY        | 复制文件                 | build的时候复制文件到image中                                 |
| ADD         | 添加文件                 | build的是偶添加文件到image中，不仅仅局限于当前的build上下文，可以来源于远程服务 |
| ENV         | 环境变量                 | 指定build时的环境变量，可以在启动容器时，通过-e覆盖，格式：ENV name=value |
| ARG         | 构建参数                 | 构建参数，只在构建的时候使用的参数，如果有ENV，那么ENV的相同名字的值始终覆盖arg的参数 |
| VOLUME      | 定义外部可以挂载的数据卷 | 指定build的image哪些目录可以启动的时候挂载到文件系统中，启动容器的时候使用-v绑定。格式：VOLUME ["目录"] |
| EXPOSE      | 暴露端口                 | 定义容器运行的时候监听的端口，启动容器的时候使用-p来绑定端口。格式：EXPOSE 8080或者EXPOSE 8080/udp |
| USER        | 指定执行用户             | 指定build或者启动的时候，用户在RUN CMD ENTRYPOINT执行的时候的用户 |
| HEALTHCHECK | 健康检查                 | 指定检测当前容器的健康检测的命令，基本没用，很多时候应用本身有健康检测机制 |
| ONBUILD     | 触发器                   | 当存在ONBUILD关键字的镜像作为基础镜像时，当执行FROM之后会执行ONBUILD命令 |
| WORKDIR     | 工作目录                 | 指定容器内部的工作目录，如果没有创建则自动创建，如果指定/使用的时绝对路径，如果不是/开头那么是在上一条workdir的路径的相对路径 |
| STOPSIGNAL  | 发送信号量到宿主机       | 该STOPSIGNAL指令设置将发送到容器的系统调用信号以退出         |
| SHELL       | 指定执行脚本的shell      | 指定RUN CMD ENTRYPOINT 执行命令时使用的shell                 |

## 3.dockerfile案例——自定义centos7镜像

要求：

- 默认登录路径为/user
- 可以使用vim

实现步骤：

- 定义父镜像
- 定义作者信息
- 执行安装vim命令
- 定义默认的工作目录
- 定义容器启动执行的命令

dockerfile编写：

```dockerfile
FROM centos:7
MAINTAINER aqi <gwq1540478105@foxmail.com>
RUN yum install -y vim
WORKDIR /usr
CMD /bin/bash
```

执行build命令：

```shell
docker build -f ./centos7_test_dockerfile -t centos7_test:1 .
```

## 4.dockerfile案例——Springboot项目

步骤：

- 编写dockerfile
  - 定义父镜像
  - 定义作者信息
  - 将jar包添加进容器
  - 定义容器启动执行的命令
- 通过dockerfile构建镜像

dockerfile编写：

```dockerfile
FROM java:8
MAINTAINER aqi
ADD zuobiji-0.0.1-SNAPSHOT.jar app.jar
CMD java -jar app.jar --spring.profiles.active=dev
```

构建镜像：

```shell
docker build -f ./zuobiji_dockerfile -t zuobiji:1 .
```

# 七、Docker Compose

## 1.服务编排

服务编排：微服务架构的应用系统中一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动启停，维护的工作量会很大。服务编排即是按照一定的业务规则批量管理容器。

## 2.Docker Compose

​		Docker Compose是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建，启动和停止。

​		使用步骤：

			1.	利用Dockerfile定义运行环境镜像
			1.	使用docker-compose.yml定义组成应用的各服务
			1.	运行docker-compose up启动应用

## 3.Docker Compose安装使用

### 3.1安装与卸载

- 安装Docker Compose

```shell
# Compose目前已经完全支持Linux、Mac Os和windows，在我们安装Compose之前，需要先安装Docker。下面我们以编译好的二进制包方式安装在Linux系统中。
curl -L "https://github.com/docker/compose/releases/download/v2.18.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
curl -L https://get.daocloud.io/docker/compose/releases/download/v2.4.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# 设置文件可执行权限
chmod +x /usr/local/bin/docker-compose
# 查看版本信息
docker-compose -version
```

- 卸载Docker Compose

```shell
# 二进制包方式安装的，删除二进制文件即可
rm /usr/local/bin/docker-compose
```

### 3.2使用docker compose编排nginx+springboot项目

- 创建docker-compose目录

```shell
mkdir ~/docker-compose
cd ~/docker-compose
```

- 编写docker-compose.yml文件

```yaml
version: "3"
services:
  nginx: # 服务名称，用户自定义
    image: nginx:1.24.0 
    ports:
      - 80:80
    volumes:
      - /root/nginx/nginx_1.24.0/html:/usr/share/nginx/html
      - /root/nginx/nginx_1.24.0/nginx.conf:/etc/nginx/nginx.conf
    privileged: true  # nginx权限问题
  mysql:
    image: mysql:5.7.41
    ports: 
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=fengweim
#   redis:
#   image: redis:latest
  zuobiji:
    image: zuobiji:latest
    build: .  #表示以当前目录下的Dockerfile开始构建镜像
    ports:
      - 11451:11451
    depends_on:
      - mysql
      #- redis
```

- 创建./nginx/conf.d目录

```shell
mkdir -p ./nginx/conf.d
```

- 在./nginx/conf.d目录下编写nginx.conf文件

```she
#user root;
worker_processes 1;
events {
	worker_connections 1024;
}
http {
	include mime.types;
	default_type application/octest-stream;
	sendfile on;
	keepalive_timeout 65;
	server {
		listen 80;
		server_name xxx.xxx.xxx.xxx;
		location / {
			root /usr/share/nginx/html;
			try_files $uri $uri/ /index.html;
			index index.html index.htm;
		}
		# 前端项目中，如果有设置默认开头/api，则设置
		#location /api/ {
		# 	proxy_pass http://xxx.xxx.xxx.xxx:xxxx;
		#	proxy_redirect default;
		#	rewrite ^/api/(.*) /$1 break;
		#}
		error_page 500 502 503 504 /50x.html;
		location = /50x.html {
			root html;
		}
	}
}
```

- 将前端Vue项目移动到/root/nginx/nginx_x.x.x/html下

- 在~/docker-compose目录下使用docker-compose启动容器

```shell
docker-compose up -d
```

- 测试访问

```http
http://xxx.xxx.xxx.xxx/hello
```

## 4.docker-compose部署前后端分离项目时遇到的坑

## 4.1编排时发生错误

执行docker-compose up -d进行编排的时候，产生以下错误：

```shell
[+] Running 0/0
 ⠋ zuobiji Pulling                                                                                                                                                                                          0.0s 
 ⠋ mysql Pulling                                                                                                                                                                                            0.0s 
error getting credentials - err: exit status 1, out: `Cannot autolaunch D-Bus without X11 $DISPLAY`
```

​	经过查询，（未验证）问题出在Linux缺少一个密码管理包`gnupg`，它用于加密，我们在登录时需要这个包将密码加密后才能完成，因此直接安装：

```shell
sudo apt install gnupg2 pass
```

### 4.2Nginx启动不了

​	在部署完后发现nginx启动不了，单独启动也没反应。

​	后来通过 docker logs -f [容器名] 才发现是配置错了。

​	nginx发生错误要去日志中查看。

​	

















