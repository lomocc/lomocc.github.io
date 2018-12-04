---
title: Kubernetes 基础知识
tags: devops
comments: true
---

# Kubernetes 基础知识

## DNS

DNS 服务器自动为 Service 生成 DNS 记录, 所以 Pod 内可以通过 `${serviceName}.${serviceNamespace}` 访问到 Service.

比如 Namespace my-ns 下有一个名为 my-service 的 Service, 则会自动生成一条 `my-service.my-ns` 的 DNS 记录。

Namespace my-ns 下的所有 Pod 应该能够通过 `my-service` 访问到这个 Service:
```shell
$ ping my-service
PING my-service.my-ns.svc.cluster.local (10.233.49.10): 56 data bytes
```

其他 Namespace 中的 Pod 可以通过 `my-service.my-ns` 访问这个 Service:
```shell
$ ping my-service.my-ns
PING my-service.my-ns.svc.cluster.local (10.233.49.10): 56 data bytes
```

这些名称查询的结果是 my-service 的 ClusterIP。

## livenessProbe 和 readinessProbe

livenessProbe 和 readinessProbe 配置用于检测应用的状态的地址和参数，以便 kubernetes 自动统计存活 pod 数量等，方便负载均衡和高可用，所以我们项目最好配置一个。

```
livenessProbe:
  httpGet:
	path: /help
	port: 5000
  initialDelaySeconds: 5
  timeoutSeconds: 3
  failureThreshold: 200
readinessProbe:
  httpGet:
	path: /help
	port: 5000
  initialDelaySeconds: 5
  timeoutSeconds: 3
```
## imagePullSecrets

```
// 创建 imagePullSecrets
$ kubectl create secret docker-registry harbor --docker-server=harbor.server.com --docker-username=usr --docker-password=pwd --docker-email=email@example.com

// 使用 imagePullSecrets
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: harbor
```
要注意 namespace 要相同

