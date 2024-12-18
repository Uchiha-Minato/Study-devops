# 使用Dockerfile定制镜像

定制镜像的本质：*定制每一层所添加的配置、文件。*

可以把每一层的修改、安装、构建等操作都写入一个脚本，用这个脚本来构建镜像。这个脚本就是`Dockerfile`。

使用`Dockerfile`，可以解决镜像构建无法重复的问题、镜像构建透明性的问题、体积的问题。

## Dockerfile

是一个文本文件，其中包含了一条一条的**指令**，每一条指令构建一层。

比如Nginx自定义的Dockerfile：

在空白目录中创建`Dockerfile`文件

```bash
mkdir mynginx
cd mynginx
touch Dockerfile
```

编辑Dockerfile

```Dockerfile
FROM nginx:1.23.2

RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```


### `FROM` 指定基础镜像

定制镜像时，基础镜像必须指定。因此在`Dockerfile`中，`FROM`是必备的，且必须是第一条指令。

除了类似`nginx`、`redis`等可以直接拿来用的服务镜像，Docker Hub还提供了一些更为基础的操作系统镜像，如`ubuntu`、`debian`、`centos`、`alpine`；也有方便于开发的语言镜像，如`node`、`openjdk`、`python`、`golang`等。

此外，还可以选择不使用任何基础镜像，即使用空镜像`scratch`。

```Dockerfile
FROM scratch
...
```

如果以 scratch 为基础镜像的话，意味着不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，*所需的一切库都已经在可执行文件里了*，因此直接 `FROM scratch` 会让镜像体积更加小巧。使用 Golang 开发的应用很多会使用这种方式来制作镜像。

### `RUN` 执行命令行命令

就跟直接在命令行中输入命令一样。

shell、exec格式：

```Dockerfile
RUN <命令>

RUN ["可执行文件", "参数1", "参数2"]
```

虽然一次RUN是执行了一次命令行命令，但是每一次RUN都会让镜像多加一层，有时这并没有任何意义。如：

```Dockerfile
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```

这所有的步骤都是同一个目的：安装redis可执行文件。因此没有必要用这么多次RUN，**这不是在写shell脚本**。

```Dockerfile
FROM debian:stretch

RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

在正确的写法中，将原本7层镜像简化成了1层，还可以看到这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了 apt 缓存文件。

镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。

### `build`镜像

```bash
docker build -t <自定义镜像名:标签> <上下文路径/URL/->

docker build -t mynginx:v1 .
```

可以清晰地看到构建过程。

![build](../cld_pic/docker_build.png)

构建成功后，按照一样的方式运行这个镜像，运行结果和在容器中修改的那一版一样。

### 构建镜像上下文 `Context`

Docker运行时(Docker Runtime)分为：

- Docker引擎(Docker Engine，服务端守护进程docker daemon)
- 客户端工具(Docker Cli)

Docker引擎提供了一组`RESTful API`，称为`Docker Remote API`。docker cli通过 **Unix套接字(/var/run/docker.sock)** 和这组API 与docker守护进程通信。

在docker服务停止时使用`dockerd --debug`

![debug](../cld_pic/docker_restful.png)

可以看到docker的RESTful API。docker守护进程进入debug模式。这时候输入`docker ps -a`和 `docker images`，可以在日志界面看到调用的API：

```bash
DEBU[2024-12-18T02:21:49.053673367+08:00] Calling HEAD /_ping                           spanID=083e710e9d6ad9a5 traceID=3ee5d3851d24e370d9554ed775e2828a
DEBU[2024-12-18T02:21:49.057752341+08:00] Calling GET /v1.45/containers/json?all=1      spanID=45538c67203cc9ed traceID=41be8011572097abe950ef2b7c17b6b0
DEBU[2024-12-18T02:22:28.696664386+08:00] Calling HEAD /_ping                           spanID=40a200801a1373d8 traceID=a5ab3b1a0a6e5e139e37f988ddd4d422
DEBU[2024-12-18T02:22:28.698387923+08:00] Calling GET /v1.45/images/json                spanID=a5049a2724c4f19e traceID=06253134d98838907bdf644cfe32c262
```

表面上看起来所有操作都在本地，实际上都是以远程调用的形式在服务器端(Docker Engine)完成各种操作。因此docker是`C/S`设计。

远程访问docker API：

![remote](../cld_pic/docker_remote.png)


在构建镜像时，除了`RUN`，还会有将本地文件添加进镜像的操作。如`ADD`、`COPY`。

因此`docker build`构建镜像，并非在本地构建，而是**在docker engine的服务端进行构建**。

