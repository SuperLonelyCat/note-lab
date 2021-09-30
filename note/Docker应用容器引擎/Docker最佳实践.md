#### 一、基本概念

**Docker的三大组件：**

##### 1 镜像（ Image）

（1）Docker镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源以及配置等文件外，还包含一些为运行时准备的一些配置参数，如匿名卷、环境变量以及用户等。镜像不包含任何动态数据，其内容在构建之后页不会被改变。

（2）Docker镜像由多层文件系统联合组成，采用分层存储架构（`Union FS技术`），方便镜像复用和定制化。镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。

（3）开发人员可通过`Dockerfile `来构建Docker镜像。

##### 2 容器（Container）

（1）镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

（2）容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。

（3）按照 Docker最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用`数据卷(Volume)`或者`绑定宿主目录`，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。

##### 3 仓库（Repository）

（1）Docker Registry是存储、分发镜像的服务。一个 **Docker Registry** 中可以包含多个 **仓库**（`Repository`）；每个仓库可以包含多个 **标签**（`Tag`）；每个标签对应一个镜像。

（2）通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签。仓库名经常以 两段式路径 形式出现，比如 `jwilder/nginx-proxy`，前者往往意味着多用户环境下的用户名，后者则往往是对应的软件名。

（3）公有Docker Registry： 官方默认的Docker Hub、Google的Google Container Registry（K8S）以及国内的网易云镜像库和阿里云镜像库等。

​          私有Docker Registry： **Harbor**

#### 二、操作指令

注：Docker在运行时分为Docker引擎（也就是服务端守护进程）和客户端工具，客户端工具通过`Docker Remote API`与Docker引擎交互。

##### 1 查看Docker信息

```shell
# 仅仅查看docker版本
docker --version

# 查看docker版本详细信息
docker version

# 查看docker服务器信息
docker info

# 查看具体命令选项
docker指令 --help
例如：docker image --help

# 查看镜像、容器以及数据卷占用的空间
docker system df
```

##### 2 使用镜像

```shell
# 拉取镜像
# 默认镜像仓库地址：Docker Hub
# 镜像仓库名是两段式：<用户名>/<软件名>:标签，默认官方用户名为library
docker image pull [镜像仓库地址] REPOSITORY
docker image pull ubuntu:18.04

# 运行镜像，如果本地镜像库不存在该镜像，自动从远程镜像库中拉取最新镜像
docker run REPOSITORY
# 运行容器，启动容器bash
# -it：i表示交互式操作；t表示终端
# bash：表示要打开的终端
# --rm：容器退出后删除，默认情况下：退出的容器不会立即删除，需手动执行docker rm指令
docker run -it --rm ubuntu:18.04 bash
# 查看当前系统版本
cat /etc/os-release
# 查看进程信息
ps
top
# 退出top
q

# 退出bash
exit

# 查看顶层镜像
docker image ls 
# 查看包括中间层在内的所有镜像
docker image ls -a
# 查看虚悬镜像(dangling image，无标签镜像)，-f：表示过滤器
docker image ls -f dangling=true
# 产生镜像ID列表
docker image ls -q
# 列出同一仓库的镜像
docker iamge ls REPOSITORY

# 清除虚悬镜像，虚悬镜像没有存在价值，可任意删除
docker image prune
# 清除所有未使用镜像，包括虚悬镜像
docker image prune -a

# 删除镜像（必须先删除运行的容器）
# IMAGE ID取前三位及以上的字符，只要足够区分与别的镜像即可
docker image rm REPOSITORY:TAG / IMAGE ID
```

##### 3 定制镜像

**注意：不要使用 `docker commit` 定制镜像，定制镜像应该使用 `Dockerfile` 来完成。**

###### 3.1 docker commit 定制镜像

**注：应用于特殊的场合，比如：被入侵后保存现场等**。

**（1）获取nginx镜像**

```shell
docker image pull nginx:1.18
```

**（2）后台启动nginx容器**

```shell
docker run --name webserver -d -p 8080:80 nginx:1.18
```

**（3）进入容器**

```shell
docker exec -it webserver bash
```

**（4）查看nginx主页内容**

```shell
cat /usr/share/nginx/html/index.html
```

**（5）覆盖nginx主页内容**

注：如果不使用卷运行一个容器的时候，任何文件修改都会被记录于容器存储层里。

**a. 修改默认初始页面内容**

```shell
# >：表示覆盖(有则覆盖，无则创建)；>>：表示追加(追加已存在的文件)

echo '<h1>Hello, Docker!!!<h1>' > /usr/share/nginx/html/index.html
```

**b. 修改默认初始页面位置**

```shell
# 在本层目录创建文件夹
mkdir one

# 创建初始文件
echo '<h1>Hi, Fang liming!!!<h1>' > /usr/share/nginx/html/one/index.html

# 修改配置文件
vi /etc/nginx/conf.d/default.conf

location / {
	root /usr/share/nginx/html/one;
	index index.html index.htm;
}
```

**注意：docker容器内部无法使用vim，安装过程如下**

```shell
# 更新包管理器
apt-get update

# 安装vim软件包
apt-get install -y vim
```

**（6）退出容器**

```shell
exit
```

**（7）将容器保存为镜像**

注：在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。

```shell
docker commit --author "FLM" --message "modify index" webserver nginx:v5
```

**（8）查看镜像历史记录**

```shell
docker history nginx:v5
```

###### 3.2 Dockerfile 定制镜像

**（1）Dockerfile文件**

```shell
# FROM：表示指定基础镜像，必须为第一条指令
FROM REPOSITORY:TAG

# WORKDIR：表示指定工作目录或者称为当前目录，以后各层的当前目录就被改为指定的目录，如该目录不存在，WORKDIR会帮你建立目录
WORKDIR /home

# COPY：复制源文件到目标目录下，使用COPY指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等
# .：表示Docker服务端上下文
COPY ./target/app.jar /home

# ADD：与COPY指令基本一样
# 如果源路径为一个tar压缩文件的话，压缩格式为gzip,bzip2以及xz的情况下，ADD指令将会自动解压缩这个压缩文件到目标路径去
ADD entrypoint.sh /

# RUN：用来执行命令行命令
# chmod +x：更改文件权限为可执行文件
RUN chmod +x /entrypoint.sh

# ENTRYPOINT表示指定容器启动程序及参数
ENTRYPOINT ["/entrypoint.sh"]
```

**（2）entrypoint.sh文件**

```shell
# #!：表示此脚本使用/bin/sh来解释执行
#!/bin/bash

cmd='java '

### 用于线上Debug调试 ###
# if -n：表示当串的长度大于0时为真(串非空)，执行then
# java -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=9999
# dt_socket：使用的通信方式
# server：是主动连接调试器还是作为服务器等待调试器连接
# suspend：是否在启动JVM时就暂停，并等待调试器连接
# address：地址和端口，两者用:连接，地址可以省略
if [[ -n ${DEBUG} ]]; then
    cmd=${cmd}'
        -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=9999
    '
fi

# java -jar app.jar：运行app.jar包
# $@：表示传递给脚本所有参数
cmd=${cmd}'
            -jar app.jar $@
'

# 输出cmd内容
echo ${cmd}

# 执行字符串形式命令
eval ${cmd}
```

##### 4 虚悬镜像

###### 4.1 顶层镜像

```shell
查看顶层镜像：docker image ls <REPOSITORY>:<TAG>

由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 <none> 的镜像，这种无标签镜像称为 dangling image，虚悬镜像

查看顶层虚悬镜像：docker image ls -f dangling=true

删除虚悬镜像：docker image prune

注：无法删除对应容器正在运行或容器处于终止态的镜像

   对于处于终止态的容器，使用 docker container prune 进行清除后，在使用 docker image prune 删除相应的镜像

查看正在运行的容器：docker container ls

查看正在运行和已终止的容器：docker container ls -a

删除虚悬镜像和未被容器使用的镜像：docker image prune -a

删除处于终止态的容器：docker container prune
```

###### 4.2 中间层镜像

```shell
查看顶层和中间层镜像：docker image ls -a <REPOSITORY>:<TAG>

使用 docker image ls -a 查看中间层镜像，中间层镜像存在许多无标签镜像，而这些无标签镜像不应该删除，否则会导致上层镜像因为依赖丢失而出错。只要删除那些依赖它们的镜像后，这些依赖的中间层镜像也会被连带删除。

删除虚悬镜像和未被容器使用的镜像：docker image prune -a
```

##### 5 操作容器

```shell
# 新建并启动容器，输出"Hello World"之后，关闭容器
docker run ubuntu:18.04 /bin/echo 'Hello World'
# 使用标签为latest的镜像后台启动容器
# -d：表示容器守护态运行
docker run -dit ubuntu 

# 查看所有容器(正在运行或已终止)，可通过STATUS查看
docker ps -a
docker container ls -a

# 查看正在运行的容器
docker container ls

# 启动已终止的容器，容器后台运行
docker container start CONTAINER ID

# 使用docker attach进入到后台运行的容器中，三位及以上字符即可，输入指令exit退出并终止容器
docker attach CONTAINER ID
# 使用docker exec进入到后台运行的容器中，三位及以上字符即可，输入指令exit退出容器，但不会终止容器
docker exec -it CONTAINER ID bash

# 终止运行中的容器
docker container stop CONTAINER ID

# 终止运行中的容器，然后在重新启动该容器
docker container restart CONTAINER ID

# 删除指定容器
# CONTAINER ID取前三位及以上的字符即可
docker container rm CONTAINER ID

# [强制]删除处于终止态的容器
docker container prune [-f]

# 容器中安装vim指令包
apt-get update
apt-get install vim
```

##### 6 访问仓库

```shell
# 查找官方仓库中的镜像
docker search centos

# 将镜像推送到Docker Hub
docker push Docker账号用户名/ubuntu:18.04
```
