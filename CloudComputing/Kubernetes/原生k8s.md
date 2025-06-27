演讲版本20250417

## workloads

### Deployment(无状态)

会保证pod一直处于预定的工作状态

滚动升级策略

### StatefulSet(有状态)

使pod有能力记住之前的状态

hostname和podName不变

### DeamonSet(守护进程集)

在每个节点上都跑一个实例

### Job && CronJob

任务/定时任务

## Pod调度

1. 以Request调度，但不能超过limit

mem超过，直接oom干掉；cpu超，会锁时间片

2. NodeSelector

打标签调度

3. NodeAffinity

节点亲和度：亲和/反亲和

4. PodAffinity

pod亲和度：让某个pod调度到另一个指定pod所在的节点上

5. Extended Resources 拓展资源

## 配置管理

### ConfigMap cm

java: application.yaml

K8s会将cm挂载到容器中的某个路径，程序去对应路径拿这个cm

### Env

环境变量

### Secret

在configmap的基础上，对值进行一个Base64的转码

## 服务发现和负载均衡

### ClusterIP

集群内通过svc IP访问，利用Kube-proxy(DaemonSet) -> Flannel

### CoreDNS

通过svc的名称访问

### NodePort

K8s分配端口，集群外能访问(端口值30000-)

### LoadBalancer

### Ingress

入项流量，配置规则，所有的流量都从Ingress进来，再由Ingress转发到集群中服务

rdse: APISIX

## 存储与数据持久化

### Volume 卷

### PV & PVC

### StorageClass & CSI

实现PV和PVC的底层逻辑 Container Storage Interface

接口能力由厂商提供

## 权限与安全

### 认证 Authentication

在apiserver启动参数中指定

1. 证书 kube-root.ca

2. 静态token

3. 用户名和密码

4. ServiceAccount

用户不是人，而是程序，每个ns有个默认的sa

k8s创建程序，程序需要读取sa中的token才能访问集群

### RBAC

Role/RoleBinding ClusterRole/ClusterRoleBinding

### SecurityContext

## 自动扩缩容

1. HPA Horizonal 水平

2. VPA Vertial 垂直

VPA是CRD资源，需要装插件

3. CA 第三方扩展，云平台提供能力

## 监控 Prometheus

## 日志 EFK

## api扩展 定制开发

1. CRD

示例程序sample-controller

2. Aggregated API

实例程序sample-apiserver
