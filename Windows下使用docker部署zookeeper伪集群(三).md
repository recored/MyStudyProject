## 使用Dockerfile构建自己的镜像
鉴于使用官方的容器缺少很多基础组件，因此我们需要使用dockerfile构建自己的镜像，并且搭配docker-compose工具进行批量化部署  
首先我们需要了解dockerfile的基本语法
FROM：选择指定的镜像  
RUN：在容器内执行对应的指令  
COPY：复制指令，从上下文目录中复制文件或者目录到容器里指定路径。  
ADD：和 COPY 的使用格类似  
CMD：类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:  
	·CMD 在docker run 时运行。  
	·RUN 是在 docker build时执行。  
ENTRYPOINT  
ENV：定义环境变量  
ARG：构建参数，与 ENV 作用一至。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。  
VOLUME：定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。  
EXPOSE：声明映射端口  
WORKDIR：指定工作目录  
USER：用于指定执行后续命令的用户和用户组  
HEALTHCHECK：用于指定某个程序或者指令来监控 docker 容器服务的运行状态。  
ONBUILD：用于延迟构建命令的执行。  
LABEL：LABEL 指令用来给镜像添加一些元数据（metadata）  

以上的语法解释含义摘抄自：  
[https://www.runoob.com/docker/docker-dockerfile.html](https://www.runoob.com/docker/docker-dockerfile.html)  
我们这次的构建所用到的并不是很多，接下会给出Dockerfile的示例
```
FROM zookeeper
RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list
RUN sed -i s@/deb.debian.org/@/mirrors.aliyun.com/@g /etc/apt/sources.list
RUN apt-get clean && apt-get update && apt-get install -y \
	vim \
	openssh-server \
	net-tools \
	inetutils-ping
```
这几句的意义分别是
1. 使用本地的zookeeper镜像
2. 执行sed命令更换apt源
3. apt-get更新以及安装vi,ssh,net-tools,ping工具
完成这些一个最简单的镜像构建就做好了，其他的命令在接下来的学习中会进一步使用。
接下来我们在命令行内本级目录下执行`docker build -t tc:v1 .`来将dockerfile内的构建生成一个名称为tc:v1的镜像  
完成后，使用`docker image ls`可以看到生成的镜像。
```
PS C:\Windows\system32> docker image ls
REPOSITORY                     TAG                                                     IMAGE ID       CREATED          SIZE
tc                             v1                                                      21ad04d8b740   18 minutes ago   320MB
```

接下来再对之前的docker-compose.yml文件做一下修改，之前该文件使用的是官方的zookeeper镜像，这里我们则改为使用我们自己制作的tc:v1镜像  

```
version: '3'

services:
    zoo1:
        image: tc:v1
        restart: always
        container_name: zoo1
        ports:
            - "2181:2181"
            - "1235:22"
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
 
    zoo2:
        image: tc:v1
        restart: always
        container_name: zoo2
        ports:
            - "2182:2181"
            - "1236:22"
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
 
    zoo3:
        image: tc:v1
        restart: always
        container_name: zoo3
        ports:
            - "2183:2181"
            - "1237:22"
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
```
为了方便后续连接xshell工具，在ports里添加了三个容器的22端口映射，以便后续连接。修改完成后执行`docker-compose up -d`即可成功生成zookeeper集群了。
## 总结
dockerfile与docker-compose.yml的配置语法有很多共通相似之处，但是这部分语法仍然需要在使用过程中才能进一步理解。为此在对docker的使用有了基本了解后，下一阶段开始编程部分，使用Django来创建个人博客，并尝试使用dockerfile将博客部署到个人服务器上。
