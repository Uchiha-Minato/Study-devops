# `kubectl api-resources`

使用此命令，可以查看当前集群支持的资源及其版本号。

![kube-apiresources](./kubectl_images/api-resources.png)

- NAME 资源名称
- SHORTNAMES 资源缩写
- APIVERSION API版本
- NAMESPACED 是否区分命名空间
- KIND 资源类型

其中APIVERSION部分，以`/`分隔，前面的为API Group，后面的为API版本号。例如：

```bash
#Pod，api组为空，即为**核心组**，api版本为v1。
pods  po  v1

#守护进程，api组为apps，api版本为v1。
daemonsets  ds  apps/v1 

#动态准入控制的mutating webhook配置，api组为admissionregistration.k8s.io，api版本为v1。
mutatingwebhookconfigurations  admissionregistration.k8s.io/v1
```

# 工作负载资源

## Pod [pods/po] 核心组/v1

**Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。**

一个Pod中可以包含多个容器，这些容器共享存储、网络、以及怎样运行这些容器的规约。 

Pod 中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。

*一个Pod中只包含一个容器 (Container) 是 K8s 的最佳实践。*

## 工作负载管理

### Deployments [deployments/deploy] apps/v1

Deployment 用于管理运行一个应用负载的一组 Pod，通常适用于无状态的负载。

一个 Deployment 为 Pod 和 ReplicaSet 提供声明式的更新能力。

### ReplicaSet [replicasets/rs] apps/v1

ReplicaSet 的作用是维持在任何给定时间运行的一组稳定的副本 Pod。 

*通常会定义一个 Deployment，并用这个 Deployment 自动管理 ReplicaSet。*

### StatefulSet [statefulsets/sts] apps/v1

StatefulSet 运行一组 Pod，并为每个 Pod 保留一个稳定的标识。 这可用于管理需要持久化存储或稳定、唯一网络标识的应用。(有状态负载)

### DaemonSet [daemonsets/ds] apps/v1

DaemonSet 定义了提供节点本地设施的 Pod。这些设施可能对于集群的运行至关重要，例如网络辅助工具，或者作为 add-on 的一部分。

DaemonSet 的一些典型用法：

- 在每个节点上运行集群守护进程
- 在每个节点上运行日志收集守护进程
- 在每个节点上运行监控守护进程

### Job [jobs] batch/v1

Job 表示一次性任务，运行完成后就会停止。

Job 会创建一个或者多个 Pod，并将继续重试 Pod 的执行，直到指定数量的 Pod 成功终止。 随着 Pod 成功结束，Job 跟踪记录成功完成的 Pod 个数。 当数量达到指定的成功个数阈值时，任务（即 Job）结束。 

    删除 Job 的操作会清除所创建的全部 Pod。 
    挂起 Job 的操作会删除 Job 的所有活跃 Pod，直到 Job 被再次恢复执行。

### CronJob [cornjobs] batch/v1

CronJob 通过重复调度启动一次性的 Job。

CronJob 用于执行排期操作，例如备份、生成报告等。 

一个 CronJob 对象就像 Unix 系统上的 crontab（cron table）文件中的一行。 它用 Cron 格式进行编写， 并周期性地在给定的调度时间执行 Job。

### ReplicationController [replicationcontrollers/rc] 核心组/v1

用于管理可水平扩展的工作负载的旧版 API。 被 Deployment 和 ReplicaSet API 取代。

确保在任何时候都有指定数量的 Pod 副本处于运行状态。

******

# Service资源

## Service [services/svc] 核心组/v1

## Endpoint [endpoints/ep] 核心组/v1

## EndpointSlice [endpointslices] discovery.k8s.io/v1

## Ingress [ingresses] networking.k8s.io/v1

## IngressClass [ingressclasses] networking.k8s.io/v1

******

# 配置和存储资源

## ConfigMap [configmaps/cm] 核心组/v1

## Secret [secrets] 核心组/v1

## Volume [volumes] 核心组/v1

## PersistVolume [persistvolumes/pv] 核心组/v1

## PersistVolumeClaim [persistvolumecclaims/pvc] 核心组/v1

## StorageClass [storageclasses/sc] storage.k8s.io/v1

******

# 身份认证、鉴权资源

## ServiceAccount [serviceaccounts/sa] 核心组/v1

## Role [roles] rbac.authorization.k8s.io/v1

## RoleBinding [rolebindings] rbac.authorization.k8s.io/v1

## ClusterRole [clusterroles] rbac.authorization.k8s.io/v1

## ClusterRoleBinding [clusterrolebindings] rbac.authorization.k8s.io/v1

******

# 扩展资源

## CustomResourceDefination [customresourcedefinations/crd] apiextensions.k8s.io

## MutatingWebhookConfiguration [mutatingwebhookconfigurations] admissionregistration.k8s.io

## ValidatingWebhookConfiguration [validatingwebhookconfigurations] admissionregistration.k8s.io

******

# 集群资源

## Event [events/v1] 核心组/v1

## Namespace [namespaces/ns] 核心组/v1

## Node [nodes/no] 核心组/v1

## APIService [apiservices] apiregistration.k8s.io/v1