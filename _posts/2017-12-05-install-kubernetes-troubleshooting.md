---
title: 部署 Kubernetes 集群过程中遇到的问题和解决方法
tag: devops
---

# 部署 Kubernetes 集群过程中遇到的问题和解决方法

## Kubernetes 不支持 swap

详情参考 issues: [Kubelet/Kubernetes should work with Swap Enabled](https://github.com/kubernetes/kubernetes/issues/53533)

解决方法:

```shell
$ swapoff -a
```

或者修改 `kubelet` 启动参数

```
# /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS=--fail-swap-on=false
```

查看是否生效, Swap 值为 0 就表示正确。

```shell
$ free -m
```
**但是这种方法重启后可能会被还原，永久解决的办法是修改文件 `/etc/fstab`，注释文件中所有包含 `swap` 单词的行**


## 端口无法访问

解决方法:

```shell
$ systemctl disable firewalld
$ systemctl stop firewalld
$ vim /etc/selinux/config // SELINUX=disabled
```

## Kubespray 清理缓存

```shell
rm -rf /tmp/node*
```

## Docker 无法拉取镜像

解决方法:

配置 [Docker 代理]({{ site.baseurl }}{% link _posts/2017-12-04-docker-proxy.md %})

## Ingress 语法改变
`Ingress` 升级到 `0.9.0` 后(https://quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.9.0), `annotations` 语法改变导致配置失效。

|||
|-|-|
|变更前|`ingress.kubernetes.io/rewrite-target: /`|
|变更后|`nginx.ingress.kubernetes.io/rewrite-target: /`|

`Annotations` 参考：
* [0.9.0-beta.17](https://github.com/kubernetes/ingress-nginx/blob/nginx-0.9.0-beta.17/docs/user-guide/annotations.md)
* [0.9.0-beta.18](https://github.com/kubernetes/ingress-nginx/blob/nginx-0.9.0-beta.18/docs/user-guide/annotations.md)(变更版本)

解决方法:

使用 `nginx.ingress.kubernetes.io/` 前缀
