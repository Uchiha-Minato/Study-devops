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
****

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

******

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

******

### `build`镜像

```bash
docker build -t <自定义镜像名:标签> <上下文路径/URL/->

docker build -t mynginx:v1 .
```

可以清晰地看到构建过程。

![build](../cld_pic/docker_build.png)

构建成功后，按照一样的方式运行这个镜像，运行结果和在容器中修改的那一版一样。

*****

### 构建镜像上下文 `Context`

Docker运行时(Docker Runtime)分为：

- Docker引擎(Docker Engine，服务端守护进程docker daemon)
- 客户端工具(Docker Cli)

Docker引擎提供了一组`RESTful API`，称为`Docker Remote API`。docker cli通过 **Unix套接字(/var/run/docker.sock)** 和这组API 与docker守护进程通信。

在*docker服务停止*时使用`dockerd --debug`

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

构建时，用户指定构建镜像上下文的路径，`docker build` 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。

Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

如果Dockerfile里这么写：
```Dockerfile
COPY ./abc.txt /app/
```

这并不是将当前目录下的abc.txt文件拷贝进容器，而是将上下文路径下的这个文件拷贝进容器。

因此，COPY 这类指令中的源文件路径都是*相对路径*。

比如：
```Bash
docker build -t abc:v1 . # 以当前目录作为上下文目录

docker build -t abc:v2 / 
# 以根目录作为上下文目录，相当于打包整个硬盘
```

一般来说，应该会将 Dockerfile 置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。

如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 `.gitignore` 一样的语法写一个 `.dockerignore`，该文件是用于剔除不需要作为上下文传递给 `Docker` 引擎的。

在默认情况下，如果不额外指定 Dockerfile ，会将上下文目录下的名为 Dockerfile 的文件作为 Dockerfile。也可使用`-f`参数指定Dockerfile。

*****

### `COPY` 复制文件

COPY 指令将从构建上下文目录中 `<源路径>` 的文件/目录复制到新的一层的镜像内的 `<目标路径>` 位置。

```Dockerfile
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
```

源路径可以有多个，也可以用通配符。

```Dockerfile
COPY hom* /app/
COPY hom?.txt /app/
```

目标路径可以是容器内的绝对路径，也可以是相对于工作目录的相对路径(Workdir)，目标路径若不存在会自动创建。

此外，使用`COPY`指令，**源文件的各种元数据都将保留**，如读写权限等。使用此指令时，也可以带上`--chown=<user>:<group>`参数设置文件所属。

如果源路径为文件夹，复制的时候不是直接复制该文件夹，而是将文件夹中的内容复制到目标路径。

****

### `ADD` 也是复制文件，功能更多

**在Docker最佳实践文档中要求，尽可能的使用`COPY`**，因为`COPY`语义明确，只是复制文件，而`ADD`包含了更为复杂的功能，其行为也不一定很清晰。

最适合使用`ADD`的情况，就是**需要自动解压缩的时候**。

如果 `<源路径>` 为一个 `tar` 压缩文件的话，压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下，`ADD` 指令将会自动解压缩这个压缩文件到 `<目标路径>` 去。

```Dockerfile
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz \
...
```

如果用`COPY`传进去需要解压缩，还得再加一层`RUN`解压缩，这时候直接使用`ADD`就会少一层。

但如果不需解压，须直接用`COPY`。

****

### `ENV` 设置环境变量

格式有两种：

```Dockerfile
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>
```

这个命令的用处很简单，就只是配置环境变量。无论后面的其他指令还是启动的应用，都可以直接使用这里的环境变量。

定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。比如在官方 `node` 镜像 Dockerfile 中，就有类似这样的代码：

```Dockerfile
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```

可以支持环境变量展开的指令： `ADD`、`COPY`、`ENV`、`EXPOSE`、`FROM`、`LABEL`、`USER`、`WORKDIR`、`VOLUME`、`STOPSIGNAL`、`ONBUILD`、`RUN`

使用环境变量，可以使Dockerfile的可重用性大大增加。

*****

### 利用上述命令和mysql官方镜像，自定义带有tls的mysql镜像

测试环境：`OpenEuler 22.04 LTS` `Docker Engine: 26.1.3`

1. 首先在docker hub上拉取官方的mysql镜像：

```bash
docker pull mysql:8.0.40
```

2. 查看虚机有没有openssl环境，没有则安装：

```bash
openssl version
yum install -y openssl
```

3. 按照下面步骤生成tls证书

**mysql只支持pem格式的证书。**

```bash
mkdir mysql-tls && cd mysql-tls

# 生成CA私钥
openssl genpkey -algorithm RSA -out ca-key.pem -aes256

# 利用私钥生成CA证书
openssl req -new -x509 -key ca-key.pem -out ca.pem -days 3650

# 生成服务器端私钥
openssl genpkey -algorithm RSA -out server-key.pem

# 生成服务器端证书请求
openssl req -new -key server-key.pem -out server-req.pem

# 生成服务器端证书
openssl x509 -req -in server-req.pem -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -days 3650

# 生成客户端私钥
openssl genpkey -algorithm RSA -out client-key.pem

# 生成客户端证书请求
openssl req -new -key client-key.pem -out client-req.pem

# 生成客户端证书
openssl x509 -req -in client-req.pem -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out client-cert.pem -days 3650

```

这时候，文件夹里应该有这些东西：

```bash
[root@openeuler mysql-tls]# ls -l
总用量 32
-rw-r--r--. 1 root root 1229 12月 19 16:57 ca.pem
-rw-r--r--. 1 root root 1675 12月 19 16:56 ca-key.pem
-rw-r--r--. 1 root root 1082 12月 19 16:58 client-cert.pem
-rw-------. 1 root root 1679 12月 19 16:58 client-key.pem
-rw-r--r--. 1 root root  948 12月 19 16:58 client-req.pem
-rw-r--r--. 1 root root 1082 12月 19 16:58 server-cert.pem
-rw-------. 1 root root 1679 12月 19 16:58 server-key.pem
-rw-r--r--. 1 root root  952 12月 19 16:57 server-req.pem
```

然后再创建一个文件夹，将下面三个文件拷进去：
```bash
mkdir mysql-server-tls

cp ca.pem mysql-server-tls/
cp server-cert.pem mysql-server-tls/
cp server-key.pem mysql-server-tls/

cd mysql-server-tls
```

4. 编写Dockerfile

```Dockerfile
FROM mysql:8.0.40

# 设置环境变量
ENV MYSQL_ROOT_PASSWORD=<密码>

# 拷贝文件进mysql容器
COPY ca-cert.pem /etc/mysql/ssl/ca.pem
COPY server-cert.pem /etc/mysql/ssl/server-cert.pem
COPY server-key.pem /etc/mysql/ssl/server-key.pem

# 修改文件权限，添加配置文件
RUN chown mysql:mysql /etc/mysql/ssl/ca.pem \
&& chown mysql:mysql /etc/mysql/ssl/server-cert.pem \
&& chown mysql:mysql /etc/mysql/ssl/server-key.pem \
&& touch /etc/mysql/conf.d/ssl.cnf \
&& echo "[mysqld]" > /etc/mysql/conf.d/ssl.cnf \
&& echo "require_secure_transport=ON" >> /etc/mysql/conf.d/ssl.cnf \
&& echo "ssl-ca=/etc/mysql/ssl/ca.pem" >> /etc/mysql/conf.d/ssl.cnf \
&& echo "ssl-cert=/etc/mysql/ssl/server-cert.pem" >> /etc/mysql/conf.d/ssl.cnf \
&& echo "ssl-key=/etc/mysql/ssl/server-key.pem" >> /etc/mysql/conf.d/ssl.cnf
```

5. 打镜像，启动

```bash
# 制作镜像
docker build -t mysql-tls:8.0.40 .

# 查看镜像
[root@openeuler mysql-server-tls]# docker images
REPOSITORY      TAG       IMAGE ID       CREATED         SIZE
mysql-tls       8.0.40    dc15c361acfc   4 hours ago     591MB
mysql           8.0.40    6c55ddbef969   2 months ago    591MB

# 启动容器
docker run -d -p 3306:3306 --name mysql-tls mysql-tls:8.0.40 

# 查看容器
[root@openeuler mysql-server-tls]# docker ps -a
CONTAINER ID   IMAGE              COMMAND                   CREATED         STATUS         PORTS                                                  NAMES
814685764f43   mysql-tls:8.0.40   "docker-entrypoint.s…"   4 seconds ago   Up 3 seconds   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql-tls

# 进入容器，输入
docker exec -it mysql-tls /bin/bash

bash-5.1# mysql -uroot -p

mysql > show variables like `%ssl%`;
```

看到以下输出：

![mysql-tls](../cld_pic/mysql-tls1.png)

将客户端私钥和证书拷出去，在Windows的MySQL Workbench中连接：

![mysql-tls2](../cld_pic/mysql-tls2.png)

配置成功。

*****
