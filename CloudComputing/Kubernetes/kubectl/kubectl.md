# kubectl

```bash
[root@openeuler02 ~]# kubectl
```
kubectl controls the Kubernetes cluster manager.

Find more information at: https://kubernetes.io/docs/reference/kubectl/

## 基础 (起始):

- `create`          从文件或标准输入(stdin)创建资源
- `expose`          将一个`replica controller` `service` `deployment`或 `Pod` 作为新的 Kubernetes 服务进行暴露。
- `run`             在集群上运行特定镜像
- `set`             为对象设置指定特性

## 基础 (中间):

- `explain`         获取一个资源的文档
- `get`             显示一个或多个资源
- `edit`            编辑服务器上的资源
- `delete`          按文件名、标准输入、资源和名称或按资源和标签选择器删除资源

## 部署:

- `rollout`         管理资源的 `rollout`
- `scale`           给 `deployment`, `replica set`,或 `replication controller` 设置新的大小
- `autoscale`       给 `deployment`, `replica set`, `stateful set`,或`replication controller` 自动设置大小

## 集群管理:

- `certificate`     修改证书资源
- `cluster-info`    显示集群信息
- `top`             显示资源占用(CPU/memory)
- `cordon`          标记节点为不可调度
- `uncordon`        标记节点为可调度
- `drain`           清空节点以准备维护
- `taint`           更新一个或者多个节点上的污点

## 问题定位以及调试:
- `describe`        显示特定资源或资源组的详细信息
- `logs`            打印 `Pod` 中容器的日志
- `attach`          挂接到一个运行中的容器
- `exec`            在某个容器中执行一个命令
- `port-forward`    将一个或多个本地端口转发到某个 `Pod`
- `proxy`           运行一个指向 Kubernetes API 服务器的代理
- `cp`              复制文件或目录进容器 或 从容器中复制出来
- `auth`            检查授权
- `debug`           为工作负载和节点的错误定位创建调试session
- `events`          列出事件

## 高级:
- `diff`            将实时版本与可能应用的版本进行比较
- `apply`           通过文件名或标准输入将配置应用于资源
- `patch`           更新资源的字段
- `replace`         用文件名或标准输入替换资源
- `wait`            实验性：等待一个或多个资源的特定条件
- `kustomize`       从目录或URL构建定制目标

## 设置:
- `label`           更新某资源上的标签
- `annotate`        更新一个资源的注解
- `completion`      输出指定shell的shell完成代码 (bash, zsh, fish, or powershell)

## 其他:
- `api-resources`   打印服务器上支持的API资源
- `api-versions`    以“group/version”的形式打印服务器上支持的API版本
- `config`          修改 `.kubeconfig` 文件
- `plugin`          提供与插件交互的实用程序
- `version`         输出客户端和服务端的版本信息

## 使用:

```bash
kubectl [flags] [options]
```

Use "`kubectl <command> --help`" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
