### 一、 Kubernetes简介

#### 1 应用部署

**问题：**

​		传统部署：部署多个应用导致资源分配问题

**解决：**

​		虚拟化部署：资源隔离，重量级

​        容器部署：资源隔离，轻量级

#### 2 容器管理

容器：打包和运行应用程序

Kubernetes：可弹性运行的分布式系统框架，用于管理容器化的工作负载和服务

Kubernetes功能：

- 服务发现与负载均衡：①可使用DNS或IP公开容器 ②可分配网络流量
- 存储编排：可选择存储系统 - 本地存储或公有云
- 自动部署与回滚
- 自动完成装箱计算：可指定容器所需CPU和内存
- 自我修复：重新启动失败的容器、替换容器以及杀死不响应的容器
- 密钥与配置管理：可存储和管理敏感信息

#### 3 集群架构

![K8S设计架构](../../image\K8S设计架构.png)

##### 3.1 控制面板组件

控制面板组件（Control Plane Components）对集群做出全局决策，以及检测和响应集群事件。

**kube-apiserver**

客户端通过 Kubernetes API 查询和操纵 Kubernetes 中对象的状态，Kubernetes API 服务器的主要实现是 kube-apiserver，它是控制面板的前端。

**etcd**

兼具一致性和高可用性的分布式键值数据库，用于保存 Kubernetes 所有集群数据。

**kube-scheduler**

负责对资源进行调度，分配某个 pod 到某个节点上。

**kube-controller-manager**

- 节点控制器（Node Controller）: 负责在节点出现故障时进行通知和响应。
- 副本控制器（Replication Controller）: 负责为系统中的每个副本控制器对象维护正确数量的 Pod。
- 端点控制器（Endpoints Controller）: 填充端点 Endpoints 对象，即加入 Service 与 Pod。
- 服务帐户和令牌控制器（Service Account & Token Controllers）: 为新的命名空间创建默认帐户和 API 访问令牌。

**cloud-controller-manager**

 仅运行特定于云平台的控制回路。

- 节点控制器（Node Controller）: 用于在节点终止响应后检查云提供商以确定节点是否已被删除。
- 路由控制器（Route Controller）: 用于在底层云基础架构中设置路由。
- 服务控制器（Service Controller）: 用于创建、更新和删除云提供商负载均衡器。

##### 3.2 Node组件

节点组件在每个节点上运行，维护运行的 Pod 并提供 Kubernetes 运行环境。

**kubelet**

保证容器都运行在 Pod 中。

**kube-proxy**

允许从集群内部或外部的网络会话与 Pod 进行网络通信。

### 二、操作过程

**Kubectl 命令行工具操作 Kubernetes 集群**

#### 1 部件简介

##### 1.1 Pod

Pod是一组（一个或多个）容器， 这些容器共享存储、网络、以及怎样运行这些容器的声明。

Pod的两种主要用法：

① 运行单个容器的 Pod

② 运行多个协同工作的容器的 Pod

使用诸如 Deployment 或 Job 这类工作负载资源来创建 Pod。

每个实例使用一个 Pod，在 Kubernetes 中通常被称为副本（Replication）， 通常使用一种工作负载资源及其控制器来创建和管理一组 Pod 副本。

##### 1.2 ReplicaSet 

确保任何时间都有指定数量的 Pod 副本在运行。

不需要操作 ReplicaSet 对象：而是使用 Deployment，并在 spec 部分定义你的应用。

##### 1.3 Deployment

Deployment为 Pod 和 ReplicaSet 提供了一个声明式定义方法，用来替代以前的 ReplicationController  来方便的管理应用。

##### 1.4 Namespace

Namespace 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。常见的 pods, services, replication controllers和deployments 等都是属于某一个 namespace 的（默认是default），而node, persistentVolumes 等则不属于任何 namespace。

Namespace常用来隔离不同的用户，比如 Kubernetes 自带的服务一般运行在kube-system namespace中。

##### 1.5 Service

将运行在一组 Pod 上的应用程序公开为网络服务。

##### 1.6 Label

标签其实就一对 key/value，被关联到对象上，比如Pod，标签的使用我们倾向于能够标示对象的特殊特点，并且对用户而言是有意义的，但是标签对内核系统是没有直接意义的。

标签可以用来划分特定组的对象。

##### 1.7 Node

Node是Pod真正运行的主机，可以是物理机，也可以是虚拟机。为了管理Pod，每个Node节点上至少要运行container runtime（比如docker）、kubelet和kube-proxy服务。

默认情况下，kubelet在启动时会向 Control Plane 注册自己，并创建资源。

#### 2 常用指令

##### 2.1 创建Pod

**（1）直接创建：使用kubectl run 指令**

```shell
kubectl run NAME --image=nginx:1.18 --replicas=3 --port=80
```

**（2）Deployment 创建 **

**① kubectl create 指令：删除创建**

```shell
kubectl create -f CMD命令行窗口相对路径 + yaml文件名称
```

**② kubectl apply 指令：更新创建**

```shell
kubectl apply -f CMD命令行窗口相对路径 + yaml文件名称
```

**nginx-deployment.yaml：**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.18
        ports:
        - containerPort: 80
```

##### 2.2 查看部署

**（1）查看Pod**

```shell
# 查看namespace-default中的所有Pod
kubectl get pods

# 查看指定namespace-NAME中的所有Pod
kubectl get pods --namespace=namespace-NAME
kubectl get pod -n namespace-NAME

# 过滤出namespace-NAME中的指定Pod
kubectl get pod -n namespace-NAME | grep pod-name(部分字符)

# 查找所有命名空间下的所有Pod
kubectl get pods --all-namespaces
```

**（2）查看Deployment**

```shell
# 查看namespace-default中的所有Deployment
kubectl get deployments

# 查看指定namespace-NAME中的所有Deployment
kubectl get deployments -n namespace-NAME

# 查看指定namespace-NAME中指定的Deployment
kubectl get deployment deployment-Name -n namespace-NAME
```

**（3）查看ReplicaSet**

```shell
# 查看namespace-default中的所有ReplicaSet
kubectl get rs
```

**（4）查看所有namespace**

```shell
kubectl get namespace
```

**（5）查看Pod日志**

```shell
kubectl logs pod-NAME
```

##### 2.3 查看历史

```shell
# 查看namespace-default中deployment-NAME版本发布历史
kubectl rollout history deployment deployment-NAME

# 查看版本详细发布信息
kubectl rollout history deployment deployment-NAME --revision=REVISION
```

##### 2.4 重新部署

```shell
# 编辑namespace-default中deployment-NAME文件（默认是yaml格式，也可指定为json格式），修改image版本，k8s自动完成部署
# 在Linux下，进入到vim编辑格式：
# 	不保存退出：Esc → :q!
# 	保存退出：Esc → :
# 在Windows下，弹出记事本，直接进行编辑
kubectl edit deploy deployment-NAME

# 编辑namespace-NAME中deployment-NAME文件
kubectl edit deploy deployment-NAME -n namespace-NAME
```

##### 2.5 连接Pod

```shell
# 通过bash命令行窗口连接Pod
kubectl exec -it pod-NAME -- bash

# 默认使用bash命令行窗口连接第一个Pod
kubectl exec -it deploy/deployment-NAME -- bash

# 通过bash访问Pod中部署的服务，nginx默认端口为80
curl http://localhost
```

##### 2.6 删除Pod

**（1）删除deployment-NAME对应的Deployment，ReplicaSet 和 Pod**

```shell
kubectl delete deployment deployment-NAME
```

##### 2.7 暴露服务

**注：公开外部IP地址以访问集群中的应用程序**

**（1）暴露Service**

```shell
kubectl expose deployment deployment-NAME --type=LoadBalancer --name=service-NAME
```

**（2）查看Service**

```shell
# 暴露服务后，可查看到EXTERNAL-IP对应的地址
kubectl get services
```

**（3）查看指定Service信息**

```shell
kubectl describe service service-NAME
```

**（4）删除指定Service**

```shell
kubectl delete service service-NAME
```

##### 2.8 可视化

打开 https://link.zhihu.com/?target=https%3A//raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc5/aio/deploy/recommended.yaml，按 Ctrl + s 将 yaml 文件保存在本地，根据如下内容修改文件

```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30000
  selector:
    k8s-app: kubernetes-dashboard

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard
```

在文件的所在文件夹打开 CMD 命令行窗口，执行如下指令

```shell
kubectl create -f recommended.yaml
```

在登录之前先用命令获取到登录所需 Token，执行如下指令

```shell
# 查看Secret
kubectl -n kubernetes-dashboard get secret

NAME                               TYPE                                  DATA   AGE
default-token-qg4xc                kubernetes.io/service-account-token   3      32s
kubernetes-dashboard-certs         Opaque                                0      32s
kubernetes-dashboard-csrf          Opaque                                1      32s
kubernetes-dashboard-key-holder    Opaque                                0      32s
kubernetes-dashboard-token-56qs5   kubernetes.io/service-account-token   3      32s

# 获取Token
kubectl describe secrets -n kubernetes-dashboard kubernetes-dashboard-token-56qs5

Name:         kubernetes-dashboard-token-56qs5
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard
              kubernetes.io/service-account.uid: 4f484662-62fc-4179-b123-6d16cc5a3143

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjllXzZ1dExySWRXWUd0WF8tUGpDVFlDNmdMVm1Hb29DQVJsRFMxVUM4NjgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi01NnFzNSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjRmNDg0NjYyLTYyZmMtNDE3OS1iMTIzLTZkMTZjYzVhMzE0MyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.HvuXdDz-3DNNgJ8C_l1Bf4nNjZaNgZ8JHAsvrgGR3zSlNUB8NqpnLlfRUzZRwDVESdolMmwlhjzmJ7wh5ep5hXYNgvQw1UWozkWCxXpOvGO2a5fSNL3dry-nZKIqjtZdYW4zlTZQY0KUZjAdy3_Z2i_yLDorLXEqzmfUdN0Ive8gQUiqkSwMINgXIjkZjkvwvw8OSyEJ_vOwnl-UaM9TiFo5EQ-vMCridzcuojG8P79pLDASnQw7Q2MH05k8NwfpiES5flVS1IFKUNrbb6e8bQIr7TFJ54m_H7Tk5vUPSAjZ-W7BfjdrhoRAzKdzd7b-IdGqkXtjgBbpAf0EVIx0Mg
```

登录 https://localhost:30000；高级 →  继续前往localhost（不安全）；选择 Token，输入token

执行下列命令，安装 Rancher

```shell
docker run --privileged -d --restart=unless-stopped -p 10001:80 -p 10002:443 rancher/rancher
```

登录 https://localhost:10002