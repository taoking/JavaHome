# 📘 **Kubernetes（K8s）快速入门分享教程（Markdown 版）**

面向对象：已掌握 Docker / Docker Compose，但未使用过 K8s 的同事\
目标：**分享完后，大家能在企业 K8s 平台上部署自己的 SpringBoot 服务**

------------------------------------------------------------------------

# 🧭 目录

1.  为什么需要 Kubernetes\
2.  Kubernetes 架构体系\
3.  Kubernetes 与 Docker Compose 的核心区别\
4.  Kubernetes 核心资源对象\
5.  Spring Boot 在 K8s 上从 0 到 1 部署\
6.  K8s 管理后台（Web UI）如何操作每个模块\
7.  常用 kubectl 命令\
8.  部署 Checklist（帮助同事独立完成部署）\
9.  Q & A

------------------------------------------------------------------------

# #️⃣ **1. 为什么需要 Kubernetes？**

## 1.1 Docker / Compose 的局限性

-   只能在单机上使用\
-   无法自动重启或健康检查\
-   不支持自动扩容/缩容\
-   配置管理能力弱\
-   发布无滚动更新\
-   网络能力有限\
-   缺少监控、负载均衡、日志体系

## 1.2 Kubernetes 的价值

K8s 提供：

-   多节点集群管理\
-   自愈（健康检查失败自动重建 Pod）\
-   滚动更新、自动回滚\
-   自动扩容（HPA）\
-   配置中心（ConfigMap / Secret）\
-   统一访问（Service + Ingress）\
-   标准化 YAML 定义\
-   强大的 UI / API / CLI 管控交互

------------------------------------------------------------------------

# #️⃣ **2. Kubernetes 架构体系**

## 2.1 总体结构

    Control Plane → Node → Pod

## 2.2 控制平面角色

-   API Server\
-   Scheduler\
-   Controller Manager\
-   ETCD

## 2.3 Node 节点角色

-   Kubelet\
-   Kube-Proxy\
-   Container Runtime

------------------------------------------------------------------------

# #️⃣ **3. K8s vs Docker Compose**

  能力       Docker Compose   Kubernetes
  ---------- ---------------- ------------------
  运行范围   单机             集群
  自愈       无               有
  扩容       手动             自动 HPA
  发布       基本无           滚动更新/回滚
  配置       env              ConfigMap/Secret

------------------------------------------------------------------------

# #️⃣ **4. 核心资源对象**

-   Pod\
-   Deployment\
-   Service\
-   Ingress\
-   ConfigMap\
-   Secret\
-   PVC/PV

------------------------------------------------------------------------

# #️⃣ **5. Spring Boot 在 Kubernetes 上部署**

## 5.1 Dockerfile

``` dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY app.jar .
ENTRYPOINT ["java","-jar","app.jar"]
EXPOSE 8080
```

## 5.2 Deployment

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
        - name: demo
          image: registry/demo:1.0
          ports:
            - containerPort: 8080
```

## 5.3 Service

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: demo-svc
spec:
  selector:
    app: demo
  type: NodePort
  ports:
    - port: 8080
      nodePort: 30080
```

## 5.4 Ingress

``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
spec:
  rules:
    - host: demo.company.com
      http:
        paths:
          - path: /
            backend:
              service:
                name: demo-svc
                port:
                  number: 8080
```

## 5.5 ConfigMap

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  SPRING_PROFILES_ACTIVE: "prod"
```

------------------------------------------------------------------------

# #️⃣ **6. 管理后台如何操作**

-   表单配置镜像/端口/副本数/探针\
-   YAML 编辑模式\
-   命令行 kubectl\
-   常见平台：Rancher、KubeSphere、ACK、自研平台

------------------------------------------------------------------------

# #️⃣ **7. kubectl 常用命令**

``` bash
kubectl get pods
kubectl logs pod
kubectl describe pod name
kubectl exec -it pod -- sh
kubectl rollout restart deploy demo
kubectl scale deploy demo --replicas=3
```

------------------------------------------------------------------------

# #️⃣ **8. 部署 Checklist**

-   镜像准备\
-   Deployment\
-   Service\
-   Ingress\
-   ConfigMap/Secret\
-   Pod 日志\
-   服务访问\
-   滚动更新\
-   自动扩容

------------------------------------------------------------------------

# #️⃣ **9. Q&A**

欢迎提问！
