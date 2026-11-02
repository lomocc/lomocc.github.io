---
title: Kubespray
tag: devops
---

# 使用 Kubespray 在 CentOS 7 部署 Kubernetes 集群

## 说明

使用 [Kubeadm]({{ site.baseurl }}{% link _posts/2017-11-23-kubeadm.md %})
部署 `Kubernetes` 操作过于复杂，需要对节点做过多的操作，因此研究了一下 [Kubespray](https://github.com/kubernetes-incubator/kubespray)

下载地址参考: [Kubespray releases](https://github.com/kubernetes-incubator/kubespray/releases)

## 环境准备

|主机名|IP|说明|
|-|-|-|
|node1| 10.82.12.47|节点1|
|node2| 10.82.12.48|节点2|
|node3| 10.82.12.49|节点3|
|node4| 10.82.12.50|节点4|
|node5| 10.82.12.51|节点5|
|操作机| 10.82.12.52|用于运行 `ansible` 命令|

如果`Node` 节点已经安装了别的版本的 `Docker` ，可能需要先卸载掉。
如果无法访问 `gcr.io`，则需要配置 [Docker 代理]({{ site.baseurl }}{% link _posts/2017-12-04-docker-proxy.md %})，或者 **使用离线镜像**

## 操作机运行

```shell
# 安装必备项

$ yum -y install ansible python-pip python-netaddr
$ pip install --upgrade Jinja2

# 免密操作

$ ssh-keygen
$ ssh-copy-id root@10.82.12.47
$ ssh-copy-id root@10.82.12.48
$ ssh-copy-id root@10.82.12.49
$ ssh-copy-id root@10.82.12.50
$ ssh-copy-id root@10.82.12.51

# 获取 kubespray

$ wget https://github.com/kubernetes-incubator/kubespray/archive/v2.4.0.zip
$ unzip v2.4.0.zip
$ cd kubespray-2.4.0

# 配置 `inventory.cfg`（也可以手动生成）

$ declare -a IPS=(10.82.12.47 10.82.12.48 10.82.12.49 10.82.12.50 10.82.12.51)
$ CONFIG_FILE=my_inventory/inventory.cfg python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

手动生成配置文件 `my_inventory/inventory.cfg`：

```shell
$ cat <<EOF > my_inventory/inventory.cfg
[all]
node1 ansible_ssh_host=10.82.12.47  # ip=10.82.12.47
node2 ansible_ssh_host=10.82.12.48  # ip=10.82.12.48
node3 ansible_ssh_host=10.82.12.49  # ip=10.82.12.49
node4 ansible_ssh_host=10.82.12.50  # ip=10.82.12.50
node5 ansible_ssh_host=10.82.12.51  # ip=10.82.12.51

[kube-master]
node1
node2
node3

[etcd]
node1
node2
node3

[kube-node]
node3
node4
node5

[k8s-cluster:children]
kube-node
kube-master

EOF
```

`etcd` 节点数量必须是奇数, 否则会出现错误 [groups.etcd|length is not divisibleby 2](https://github.com/kubernetes-incubator/kubespray/blob/f9b68a5d17721496c92000881ce31233903259ba/roles/kubernetes/preinstall/tasks/verify-settings.yml#L50)

官方推荐使用 3/5/7 个节点，容错能力更强

## 其他可选配置

```shell
$ vi my_inventory/group_vars/k8s-cluster.yml

# 允许 kubernetes dashboard 常规认证方式
kube_basic_auth: true

# 使用 flannel 作为网络插件
- kube_network_plugin: calico
+ kube_network_plugin: flannel
```

## 开始部署

* 部署集群

```shell
$ ansible-playbook -i my_inventory/inventory.cfg cluster.yml -u root -b
```

* 添加节点

```shell
$ ansible-playbook -i my_inventory/inventory.cfg scale.yml -u root -b
```

* 删除集群

```shell
$ ansible-playbook -i my_inventory/inventory.cfg reset.yml -u root -b
```

## Docker 使用离线镜像

* 使用以下命令来保存提前下载的镜像

```shell
# 以下是 v1.9.2 的镜像，保存到 kubernetes-images.tar

$ images=(
quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.11.0
quay.io/coreos/hyperkube:v1.9.2_coreos.0 
quay.io/calico/node:v2.6.2
quay.io/calico/cni:v1.11.0
quay.io/calico/ctl:v1.6.1
quay.io/calico/routereflector:v0.4.0
quay.io/coreos/etcd:v3.2.4
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.8
gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.8
gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.8
gcr.io/google_containers/cluster-proportional-autoscaler-amd64:1.1.2
gcr.io/google_containers/pause-amd64:3.0
gcr.io/google_containers/kubernetes-dashboard-amd64:v1.8.1
gcr.io/google_containers/kubernetes-dashboard-init-amd64:v1.0.1
); \
for image in ${images[@]}; do docker pull $image; done; \
docker save ${images[@]} > kubernetes-images.tar
```

* 将 `kubernetes-images.tar` 复制到目标主机，使用以下命令来加载提前下载的镜像

```shell
docker load -i kubernetes-images.tar
```

* 为 `ansible-playbook` 添加参数 `--skip-tags=download`，跳过镜像下载阶段：

```shell
ansible-playbook -i my_inventory/inventory.cfg cluster.yml -u root -b --skip-tags=download
```
