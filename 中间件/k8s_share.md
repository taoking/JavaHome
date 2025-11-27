# 📘 Kubernetes（K8s）基础分享

> 面向公司内部开发 / 测试 / 运维同学的入门分享，时长建议 45–60 分钟。  
> 目标：听完之后，每个人都能“说清楚概念 + 看懂 YAML + 自己把一个服务部署到 K8s 上”。

---

## 一、分享目标 & 适用人群

**分享目标**
- 理解为什么需要 K8s，以及它大概解决什么问题。
- 掌握 K8s 的核心概念与常用资源类型。
- 能从 0 写出一个简单的 Deployment + Service 并在集群运行。
- 知道基本的排错思路和公司内部推荐实践。

**适用人群**
- 日常编写服务端 / 后端应用的开发同学。
- 需要在 K8s 环境验证功能的测试同学。
- 负责环境维护、发布上线的运维 / SRE 同学。

---

## 二、为什么会出现 Kubernetes？

**1. 传统部署方式的痛点**
- 应用直接部署在物理机 / 虚拟机上：
  - 环境不一致：依赖库版本、系统配置难以统一。
  - 资源利用率低：一台机器上只敢跑少量服务，浪费 CPU / 内存。
  - 部署和回滚麻烦：靠脚本或人工操作，容易出错。

**2. 容器带来的变化**
- 打包：应用 + 运行时 + 依赖一起打包成镜像，做到“构建一次，到处运行”。
- 隔离：不同应用之间通过容器隔离，减少相互影响。
- 启动快：容器秒级拉起，适合弹性扩缩容。

**3. 有了容器还不够：为什么需要 K8s？**
- 单纯有 Docker 以后，还会遇到：
  - 有很多容器要管理：放在哪台机器？怎么分配资源？
  - 容器挂掉怎么办：谁来拉起来、怎么保证副本数？
  - 外部怎么访问容器：IP 不固定、端口多且混乱。
  - 多环境、多团队：如何隔离、如何统一配置？
- Kubernetes 的核心价值：  
 通过**声明式配置**和一系列控制器，自动地帮我们做**调度、扩缩容、自愈、服务发现和配置管理**。

---

## 三、Kubernetes 整体架构概览

可以把 K8s 理解成一个“集群操作系统”：

```text
┌─────────────────────────┐
│      控制面 Control Plane │（负责“大脑”和控制） 
│  - API Server            │
│  - Scheduler             │
│  - Controller Manager    │
│  - etcd                  │
└─────────────┬───────────┘
              │
      （通过 API 管理）
              │
┌─────────────┴───────────┐
│      工作节点 Node       │（真正跑容器的“机器”）
│  - kubelet               │
│  - kube-proxy            │
│  - Container Runtime     │
└─────────────────────────┘
```

**关键点理解：**
- `Control Plane`：整个平台的“大脑”和“控制中心”，负责接收我们的配置、做调度和状态维护。
- `Node`：实际承载 Pod / 容器运行的工作节点，可以是虚机、物理机或云主机。
- `etcd`：分布式键值存储，保存 K8s 集群的所有状态（配置和元数据）。

---

## 四、核心概念快速扫盲

可以用“一句话”把这些概念串起来：

> 在一个 **Cluster** 中，我们按 **Namespace** 做隔离，在多个 **Node** 上运行 **Pod**。  
> 我们用 **Deployment** 等控制器管理 Pod 的副本和升级，用 **Service / Ingress** 提供稳定访问入口，  
> 用 **ConfigMap / Secret / Volume / PV / PVC** 管理配置和存储，用 **Label / Selector** 把所有资源“串起来”。

### 1. Cluster / Namespace / Node

- **Cluster（集群）**
  - 一组运行着 K8s 组件的机器的集合，是整个系统的边界。
  - 一般我们会按环境划分：测试集群、预发集群、生产集群等。

- **Namespace（命名空间）**
  - 集群内的逻辑隔离单元，类似一个“虚拟子集群”。
  - 常见用途：按环境（`dev/test/prod`）、按团队或按业务线划分。

- **Node（节点）**
  - 运行 Pod 的工作机器，可以理解为“宿主机”。
  - 每个节点上运行 `kubelet`（负责和控制面通信）和 `kube-proxy`（负责网络）。

### 2. Pod：最小调度单位

- 一个 Pod 是一组一同运行在同一 Node 上的容器集合：
  - 共享同一个 IP 地址和端口空间。
  - 共享存储卷（Volume）。
- 常见模式：
  - 一个 Pod 一个主容器（最常见）。
  - Sidecar 模式：主容器 + 辅助容器（如日志收集、代理等）。
- 注意：**Pod 是易失的**，不直接手工管理单个 Pod，而是通过更高层的控制器来管理。

### 3. 控制器：Deployment / StatefulSet / DaemonSet / Job

- **Deployment**
  - 最常用的无状态应用控制器。
  - 功能：管理 Pod 副本数、滚动升级、回滚。
  - 典型场景：Web 服务、API 服务。

- **StatefulSet**
  - 面向有状态应用（通常需要稳定网络标识 & 持久存储）。
  - Pod 有固定的序号和稳定名称，如 `xxx-0`、`xxx-1`。
  - 典型场景：数据库、某些中间件组件。

- **DaemonSet**
  - 确保每个（或部分）节点上都有一个 Pod 运行。
  - 典型场景：日志采集 Agent、监控 Agent。

- **Job / CronJob**
  - Job：一次性任务，执行完成后退出。
  - CronJob：周期性任务，类似 Linux 的 crontab。

### 4. Service：服务发现与负载均衡

- Pod 的 IP 会变化，不能直接依赖 Pod IP。
- Service 通过 **Label Selector** 找到一组 Pod，给这组 Pod 提供：
  - 稳定的虚拟 IP（ClusterIP）。
  - 在这些 Pod 之间做负载均衡。

常见类型：
- `ClusterIP`：默认类型，用于集群内部访问。
- `NodePort`：在每个节点打开一个固定端口，对集群外暴露。
- `LoadBalancer`：在云环境中创建云厂商的负载均衡器。

### 5. Ingress：统一的 HTTP/HTTPS 入口

- Ingress 提供了 HTTP/HTTPS 层的路由能力：
  - 按域名路由：`api.xxx.com`、`web.xxx.com`。
  - 按路径路由：`/api` 转发到服务 A，`/web` 转发到服务 B。
  - 可以做 TLS/HTTPS 终止。
- Ingress 本身只是一组规则，需要对应的 **Ingress Controller**（如 Nginx Ingress、Traefik 等）实现。

### 6. 配置与密钥：ConfigMap / Secret

- **ConfigMap**
  - 保存非敏感配置信息，如应用配置、开关、环境变量等。
  - 可通过环境变量或挂载文件方式注入到 Pod。

- **Secret**
  - 保存敏感数据，如密码、Token、证书等。
  - 只在需要的 Pod 中挂载 / 注入，并配合权限控制。

### 7. 存储：Volume / PV / PVC

- Volume（卷）
  - 定义在 Pod 内部，生命周期随 Pod。
  - 用于容器间共享数据或临时数据。

- PersistentVolume（PV）/ PersistentVolumeClaim（PVC）
  - PV：集群级别的存储资源，通常由运维提供（NFS、云硬盘等）。
  - PVC：应用侧对存储的“申请”，描述需要多大、什么访问模式。
  - Pod 通过 PVC 绑定到具体的 PV，做到“存储与 Pod 解耦”。

### 8. Label / Selector / Annotation

- Label：附加在资源上的键值对，如：
  - `app=order-service`、`env=prod`、`team=payment`
- Selector：基于 Label 的筛选条件：
  - Service / Deployment / HPA 等都会通过 Selector 找到相关资源。
- Annotation：非结构化备注信息，一般给工具或运维使用，不参与筛选。

---

## 五、从 0 到 1：部署一个最简单的 Web 应用

目标：在 K8s 集群里跑一个 Nginx 服务，并通过 Service 访问。

### 1. 前置条件
- 已有可访问的 K8s 集群（本地 kind / minikube，或公司内部集群）。
- `kubectl` 已配置好上下文，可以正常执行 `kubectl get nodes`。

### 2. 编写 Deployment + Service（示例）

创建一个文件 `nginx-demo.yaml`：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
  labels:
    app: nginx-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-demo-svc
spec:
  type: NodePort
  selector:
    app: nginx-demo
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

### 3. 部署与验证

```bash
# 部署
kubectl apply -f nginx-demo.yaml

# 查看 Deployment / Pod / Service
kubectl get deploy,pod,svc

# 查看某个 Pod 的详细信息
kubectl describe pod <pod-name>

# 访问（NodePort）
curl http://<任意节点IP>:30080
```

通过这个 Demo，可以直观地理解：
- Deployment 如何管理多副本 Pod。
- Service 如何给一组 Pod 提供稳定访问入口。
- YAML 的核心结构和字段。

---

## 六、常用 `kubectl` 命令速查

> 建议在 PPT 中单独放一页，供开发 / 排错时参考。

- 集群与资源查看
  - `kubectl get nodes`
  - `kubectl get ns`
  - `kubectl get pod -A`（查看所有命名空间 Pod）
  - `kubectl get deploy,svc,ingress`

- 详情与事件
  - `kubectl describe pod <pod-name>`
  - `kubectl get events --sort-by=.lastTimestamp`

- 日志与进入容器
  - `kubectl logs <pod-name>`
  - `kubectl logs -f <pod-name> -c <container-name>`
  - `kubectl exec -it <pod-name> -- /bin/sh`

- 部署与回滚
  - `kubectl apply -f xxx.yaml`
  - `kubectl delete -f xxx.yaml`
  - `kubectl rollout status deployment <name>`
  - `kubectl rollout undo deployment <name>`

- 资源管理
  - `kubectl scale deployment <name> --replicas=4`
  - `kubectl top pod`（需要 metrics-server）

---

## 七、常见问题与排错思路

适合作为分享中互动部分，可以结合实际故障案例讲解。

### 1. Pod 一直是 Pending

常见原因：
- 集群资源不足：CPU / 内存不够。
- 节点选择约束太严格：节点亲和 / 污点容忍配置导致排不进去。
- PVC 未绑定：有持久化存储需求但 PVC 无法绑定 PV。

排查步骤：
- `kubectl describe pod <pod>` 看调度事件和报错信息。
- `kubectl get node` 看资源是否充足。
- 如涉及 PVC：`kubectl get pvc,pv`。

### 2. Pod CrashLoopBackOff / Error

常见原因：
- 应用自身异常退出，如配置错误、依赖无法连接。
- 健康检查配置不合理导致频繁重启。

排查步骤：
- `kubectl logs <pod>` 看容器日志。
- 查看 liveness / readiness 配置，放宽阈值验证。
- 在本地/测试环境用相同镜像跑一遍复现问题。

### 3. ImagePullBackOff / ErrImagePull

常见原因：
- 镜像地址写错、Tag 不存在。
- 私有仓库认证失败，没有配置 `imagePullSecrets`。

排查步骤：
- `kubectl describe pod <pod>` 看拉取镜像的报错信息。
- 确认镜像仓库地址、项目名、Tag 等是否正确。
- 如是私有仓库，检查 Secret 是否存在并已配置到 Pod。

### 4. Service 访问不通

常见原因：
- Service 的 selector 与 Pod 的 label 不匹配，导致找不到后端 Pod。
- 容器内部真实监听端口与 Service 配置的 `targetPort` 不一致。
- 跨命名空间访问时，使用的 DNS 名称不正确。

排查步骤：
- `kubectl get endpoints <svc>` 看后端是否有 Pod。
- 进入 Pod 内使用 `curl localhost:<port>` 验证应用自身是否可用。
- 在同命名空间 Pod 中访问 `http://<svc-name>:<port>` 进行排查。

### 5. Ingress 不生效

常见原因：
- 集群未正确安装 Ingress Controller。
- Ingress 资源中的域名未正确解析到入口 IP。
- Ingress 规则配置与 Service 名称 / 端口不匹配。

排查步骤：
- `kubectl get pod -n <ingress-namespace>` 检查 Ingress Controller 状态。
- `kubectl describe ingress <name>` 查看事件与路由规则。
- 使用 `curl -H "Host: xxx.com" http://<ingress-ip>/path` 进行验证。

---

## 八、公司内部推荐实践（可根据实际情况调整）

> 这一部分适合结合公司现有平台 / 规范进行二次补充。

- 命名与分组
  - 按环境：`dev` / `test` / `staging` / `prod` 使用独立 Namespace 或独立集群。
  - 统一 Label 规范：`app`、`env`、`team`、`owner` 等，方便运维和观测。

- 配置管理
  - 所有可变配置放到 ConfigMap / Secret 中，不写死在镜像内。
  - 为不同环境准备不同的配置文件，避免手工改 YAML。

- 资源与伸缩
  - 为 Pod 设置合理的 `resources.requests` 和 `resources.limits`，避免“谁都抢不到资源”或“有人把机器吃满”。
  - 核心服务启用 HPA，根据 CPU / QPS / 自定义指标自动扩缩容。
  - 对核心业务增加 PodDisruptionBudget，保证升级 / 维护时可用实例数。

- 安全与权限
  - 尽量不要在容器中使用 root 用户运行应用。
  - Secret 按最小权限原则使用，不在无关 Pod 中挂载。
  - 配合 RBAC 控制不同角色对集群的访问权限。

- 发布与回滚
  - 统一通过 CI/CD 平台部署应用，禁止直接手动改线上 Deployment。
  - 重要变更采用灰度发布：先在部分实例 / 流量上验证，再全量发布。
  - 使用 Deployment 的滚动升级与回滚能力，快速恢复到上一版本。

---

## 九、参考资料（推荐会后深入阅读）

- 官方文档
  - Kubernetes 官网主页：<https://kubernetes.io/>
  - Kubernetes 官方文档（英文）：<https://kubernetes.io/docs/home/>
  - Kubernetes 中文文档（社区翻译）：<https://kubernetes.io/zh-cn/docs/home/>

- 容器基础
  - Docker 入门指南：<https://docs.docker.com/get-started/>

- 进阶书籍（可作为读书会素材）
  - 《Kubernetes in Action》
  - 《Kubernetes 权威指南》
  - 《云原生应用架构实践》

> 建议：分享结束后将本 Markdown 与 Demo YAML 示例一并放到内部代码仓库，后续新同学可以直接跟着文档练习与复现。
