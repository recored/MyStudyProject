## 镜像下载
首先执行如下命令:
```
docker pull zookeeper
```
可以下载官方最新的zookeeper镜像。
当出现如下结果时, 表示镜像已经下载完成了:
```
PS C:\Windows\system32> docker pull zookeeper
Using default tag: latest
latest: Pulling from library/zookeeper
33847f680f63: Pull complete
47d5ef013a61: Pull complete
0e6aa028937a: Pull complete
cabac214cb3b: Pull complete
170911c9c78e: Pull complete
3808da46857f: Pull complete
d12fc8641aad: Pull complete
dce3cb1dac87: Pull complete
Digest: sha256:d70d4bfa7e1ada5d4247ad9f482d2ed191eb01dddd8da3c14234a418a22498f8
Status: Downloaded newer image for zookeeper:latest
docker.io/library/zookeeper:latest
```
## ZK 镜像的基本使用
### 启动zk镜像：
```
PS C:\Windows\system32> docker run --name my_zookeeper -d zookeeper:latest
```
这个命令会在后台运行一个 zookeeper 容器, 名字是 my_zookeeper, 并且它默认会导出 2181 端口.
接着我们使用:
```
PS C:\Windows\system32> docker logs -f my_zookeeper
```
这个命令查看 ZK 的运行情况, 输出类似如下内容时, 表示 ZK 已经成功启动了:
```
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
...
```
### 使用 ZK 命令行客户端连接 ZK
我们可以通过 Docker 的 link 机制来对这个 ZK 容器进行访问. 执行如下命令:
```
docker run -it --rm --link my_zookeeper:zookeeper zookeeper zkCli.sh -server zookeeper
```
这个命令的含义是:

启动一个 zookeeper 镜像, 并运行这个镜像内的 zkCli.sh 命令, 命令参数是 "-server zookeeper"
1.将我们先前启动的名为 my_zookeeper 的容器连接(link) 到我们新建的这个容器上, 并将其主机名命名为 zookeeper
2.当我们执行了这个命令后, 就可以像正常使用 ZK 命令行客户端一样操作 ZK 服务了.
3.在zookeeper客户端下我们可以查看到基本路径.
```
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: zookeeper(CONNECTED) 0] ls /
[zookeeper]
[zk: zookeeper(CONNECTED) 1] ls /zookeeper
[config, quota]
[zk: zookeeper(CONNECTED) 2]

```
## ZK 集群的搭建
我们的目标是创建在本地docker容器下创建一个zookeeper的伪集群，这里可以直接使用 docker-compose 来启动 ZK 集群.
首先创建一个名为 docker-compose.yml 的文件, 其内容如下:
```
version: '2'
services:
    zoo1:
        image: zookeeper
        restart: always
        container_name: zoo1
        ports:
            - "2181:2181"
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
 
    zoo2:
        image: zookeeper
        restart: always
        container_name: zoo2
        ports:
            - "2182:2181"
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
 
    zoo3:
        image: zookeeper
        restart: always
        container_name: zoo3
        ports:
            - "2183:2181"
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
```
这个配置文件会告诉 Docker 分别运行三个 zookeeper 镜像, 并分别将本地的 2181, 2182, 2183 端口绑定到对应的容器的2181端口上.
ZOO_MY_ID 和 ZOO_SERVERS 是搭建 ZK 集群需要设置的两个环境变量, 其中 ZOO_MY_ID 表示 ZK 服务的 id, 它是1-255 之间的整数, 必须在集群中唯一. ZOO_SERVERS 是ZK 集群的主机列表..
接着我们在 docker-compose.yml 当前目录下运行:
这里我放在D盘目录下：
```
PS D:\work-notes\ZK> ls
目录: D:\work-notes\ZK
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2021/7/23      0:59            917 docker-compose.yml
```
在该目录执行docker-compose up
```
PS D:\work-notes\ZK>docker-compose up
```
即可启动 ZK 集群了.
执行上述命令成功后, 接着在另一个终端中运行 docker-compose ps 命令可以查看启动的 ZK 容器:
```
D:\work-notes\ZK>docker-compose ps
NAME                COMMAND                  SERVICE             STATUS              PORTS
zoo1                "/docker-entrypoint.…"   zoo1                running             0.0.0.0:2181->2181/tcp, :::2181->2181/tcp, 2888/tcp, 3888/tcp, 8080/tcp
zoo2                "/docker-entrypoint.…"   zoo2                running             0.0.0.0:2182->2181/tcp, :::2182->2181/tcp, 2888/tcp, 3888/tcp, 8080/tcp
zoo3                "/docker-entrypoint.…"   zoo3                running             0.0.0.0:2183->2181/tcp, :::2183->2181/tcp, 2888/tcp, 3888/tcp, 8080/tcp
```
## 使用 Docker 命令行客户端连接 ZK 集群
通过 docker-compose ps 命令, 我们知道启动的 ZK 集群的三个主机名分别是 zoo1, zoo2, zoo3, 因此我们分别 link 它们即可:

```
docker run -it --rm \
        --link zoo1:zk1 \
        --link zoo2:zk2 \
        --link zoo3:zk3 \
        --net zktest_default \
        zookeeper zkCli.sh -server zk1:2181,zk2:2181,zk3:2181
```

尝试link时发现docker出现报错：
```
docker: Error response from daemon: network zktest_default not found.
```
其实这里是需要将三个zk全部link到zktest_default这个目录下，因此我们还需要创建zktest_default网络

执行`docker network create zktest_default`来进行链接
```
PS D:\work-notes\ZK> docker network create zktest_default
3e2c6afade86e0a9104d5e8e820d649be98c170a5b3e06e58bf894a06d9d0212
PS D:\work-notes\ZK>
```
这里我们已经创建成功，就可以重新进行链接了。

但是此时我们可以发现并不需要link到单独的网络，zk已经能够互相连接并且组成集群了。
原因是当我们使用docker-compose.yml来部署容器时就已经将容器部署在同一套网络内部。
进入docker命令行执行命令会发现ZK的节点状态已经起来了
```
# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: leader
```
## 未完成的部分
1.使用过程中发现docker自带的客户端工具非常不方便，需要映射ssh端口以便通过外部ssh进行访问。

2.容器本身的环境需要额外安装ping工具,ifconfig,vim等进行必要的查询以及配置，需要更新国内源加快更新速度。

3.多个容器下的管理以及命令执行，此时k8s就比较重要，学习使用k8s以减少重复工作。
