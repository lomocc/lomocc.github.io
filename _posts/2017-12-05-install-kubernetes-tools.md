---
title: Kubernetes 安装 Ingress / Dashboard 等常用工具
tag: devops
---

# Kubernetes 安装 Ingress / Dashboard 等常用工具

## 安装 `Ingress`

### 基本命令

```shell
$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/namespace.yaml \
    | kubectl apply -f -

$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/default-backend.yaml \
    | kubectl apply -f -

$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/configmap.yaml \
    | kubectl apply -f -

$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/tcp-services-configmap.yaml \
    | kubectl apply -f -

$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/udp-services-configmap.yaml \
    | kubectl apply -f -
```

### 不使用 `RBAC` 安装

```shell
$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/without-rbac.yaml \
    | kubectl apply -f -
```

### 使用 `RBAC` 安装

[RBAC](https://github.com/kubernetes/ingress-nginx/blob/master/deploy/rbac.md)

```shell
$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/rbac.yaml \
    | kubectl apply -f -

$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/with-rbac.yaml \
    | kubectl apply -f -
```

### 暴露端口

使用 [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport):

```shell
$ curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml \
    | kubectl apply -f -
```

备注：默认自动分配 30000+ 端口号，可以使用 nodePort 参数来指定对外暴露端口。

## 创建 `Dashboard`

```shell
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

用于认证服务的 rbac `kubernetes-dashboard-rbac.yaml`

```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
```

用于暴露服务的 Ingress `kubernetes-dashboard-ing.yaml`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  rules:
  - host: k8s.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kubernetes-dashboard
          servicePort: 80
```
