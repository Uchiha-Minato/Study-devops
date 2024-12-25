# 使用容器

容器是独立运行的一个或一组应用，以及它们的运行态环境。

列出容器：
```bash
docker ps # 列出运行的
docker ps -a #列出所有容器
```

## 启动容器

两种方式：
- 基于镜像新建并启动
- 将终止状态的容器重启

### 新建并启动

```bash
docker run [options] <image> [command] [arg...]
```

当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从 registry 下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

`docker run` 常用选项说明

|选项|备注|举例|
|--|--|--|
|-d|后台运行容器并返回容器id|--|
|-it|交互式运行容器，分配一个伪终端|`docker run -it ubuntu /bin/bash`|
|--name|给容器指定一个名称|`--name mysql-tls`|
|-p|端口映射|`-p 30306:3306`|
|-v|挂载卷|`-v 宿主机路径:容器内路径`|
|--rm|容器停止后自动删除容器|--|
|--env,-e|设置环境变量|`--env MYSQL_ROOT_PASSWORD=XXX`|
|--network|指定容器的网络模式|--|
|--restart|指定容器的重启策略|`--restart always`|
|-u|指定用户|`-u root`|

### 重启容器

```bash
docker start <container>
# 或
docker restart <container>
```

****

## 停止容器

```bash
docker stop <container>
```

****

## 进入容器

```bash
docker exec [options] <container> <command> [arg...]

# 不推荐
docker attach [options] <container>
```

`docker attach`是将本地标准输入、输出和错误流附加到正在运行的容器。**如果从这个stdin中exit，会使容器停止。**

`docker exec`在运行的容器中执行命令，exit不会使容器停止。

常用选项

|选项|备注|举例|
|--|--|--|
|-d|后台运行|--|
|-e,--env|配置环境变量|--|
|-h,--help|帮助|--|
|-it|交互式运行容器，分配一个伪终端|`docker exec -it mysql /bin/bash`|
|-u,--user|设置用户|--|
|-w,--workdir|设置容器内工作目录|--|

****

## 导出/导入容器快照

```bash
# 导出
docker export [options] container

# 导入
docker import [options] file|URL|- [repository[:TAG]]
```

导出选项 `-o <filename>` 输出到文件以代替标准输出。

```bash
root@openeuler container]# docker ps -a 
CONTAINER ID   IMAGE              COMMAND                   CREATED        STATUS       PORTS                                                  NAMES
f8d3519e9d66   mysql-tls:8.0.40   "docker-entrypoint.s…"   23 hours ago   Up 5 hours   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql-tls
[root@openeuler container]# docker export mysql-tls -o mysql-tls.tar
[root@openeuler container]# ls -lh
总用量 553M
-rw-------. 1 root root 553M 12月 25 10:18 mysql-tls.tar
```

`docker import`从容器快照文件中再导入为镜像。

```bash
sysadm@dev:~/Documents$ docker import mysql-tls.tar mysql-tls:8.0.40
sha256:c9bff017638869a1a117739fc6691d2b7697f997d937bfbac10808a0f592a9c6
sysadm@dev:~/Documents$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
mysql-tls    8.0.40    c9bff0176388   2 minutes ago   562MB
```

既可以使用 `docker load` 来导入镜像存储文件到本地镜像库，也可以使用 `docker import` 来导入一个容器快照到本地镜像库。

这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时**可以重新指定标签等元数据信息**。

****

## 删除容器

使用`docker rm`删除一个或多个容器。

```bash
docker rm [options] <container> [container...]
```
删除选项

|选项|备注|
|--|--|
|-f,--force|强制删除正在运行的容器，使用`SIGKILL`|
|-l,--link|删除指定的链接|
|-v,--volumes|删除与容器关联的匿名卷|

删除所有已停止的容器

```bash
docker container prune
```
