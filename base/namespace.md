# Namespace(命名空间)

命名空间(Namespace)是一种资源隔离机制，将同一集群中的资源划分为相互隔离的组。
命名空间可以在多个用户之间划分集群资源（通过[资源配额](https://kubernetes.io/zh-cn/docs/concepts/policy/resource-quotas/)）。

- 例如我们可以设置开发、测试、生产等多个命名空间。

同一命名空间内的资源名称要唯一，但跨命名空间时没有这个要求。
命名空间作用域仅针对带有名字空间的对象，例如 Deployment、Service 等。
这种作用域对集群访问的对象不适用，例如 StorageClass、Node、PersistentVolume 等。

```sh
# 查看命名空间 缩写 ns
$ kubectl get namespace
NAME              STATUS   AGE
default           Active   4d15h
kube-system       Active   4d15h
kube-public       Active   4d15h
kube-node-lease   Active   4d15h
```

---

## Kubernetes 会创建四个初始命名空间：

- default 默认的命名空间，不可删除，未指定命名空间的对象都会被分配到 default 中。
- kube-system Kubernetes 系统对象(控制平面和 Node 组件)所使用的命名空间。
- kube-public 自动创建的公共命名空间，所有用户（包括未经过身份验证的用户）都可以读取它。通常我们约定，将整个集群中公用的可见和可读的资源放在这个空间中。
- kube-node-lease [租约（Lease）](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/lease-v1/)对象使用的命名空间。每个节点都有一个关联的 lease 对象，lease 是一种轻量级资源。lease 对象通过发送心跳，检测集群中的每个节点是否发生故障。

<p class="r">使用 kubectl get lease -A 查看 lease 对象</p>

- kubenetes 的组件都运行在 kube-system 命名空间中, 自己部署的在 default 目录下

```sh
$ kubectl get pod -A
NAMESPACE     NAME                                     READY   STATUS      RESTARTS         AGE
kube-system   helm-install-traefik-crd-bjf84           0/1     Completed   0                4d15h
kube-system   helm-install-traefik-bcrr6               0/1     Completed   5                4d15h
kube-system   svclb-traefik-c090484e-hwms9             2/2     Running     2 (3d11h ago)    4d15h
kube-system   svclb-traefik-c090484e-4rlrg             2/2     Running     2 (3d11h ago)    4d15h
kube-system   coredns-77ccd57875-rf6s4                 1/1     Running     10 (3d11h ago)   4d15h
kube-system   svclb-traefik-c090484e-cnsgm             2/2     Running     2 (3d11h ago)    4d15h
kube-system   local-path-provisioner-957fdf8bc-dw92l   1/1     Running     0                4d15h
kube-system   metrics-server-648b5df564-tnzrm          1/1     Running     30 (3d11h ago)   4d15h
kube-system   traefik-64f55bb67d-bhb49                 1/1     Running     28 (3d11h ago)   4d15h
default       nginx-deploy-5964889c54-fkd8t            1/1     Running     0                152m
default       nginx-deploy-5964889c54-lnd79            1/1     Running     0                152m
default       nginx-deploy-5964889c54-tfdf7            1/1     Running     0                152m
```

- 新版本 apiserver 也是一个 lease 对象

```sh
$ kubectl get lease -A
NAMESPACE         NAME                                   HOLDER                                                                      AGE
kube-node-lease   k3d-demo-agent-0                       k3d-demo-agent-0                                                            4d15h
kube-node-lease   k3d-demo-agent-1                       k3d-demo-agent-1                                                            4d15h
kube-node-lease   k3d-demo-server-0                      k3d-demo-server-0                                                           4d15h
kube-system       apiserver-tbovar5ze2pqx4h6gvtcm557sm   apiserver-tbovar5ze2pqx4h6gvtcm557sm_8167404a-b62f-45fb-81e8-72eddaf1aaf8   4d15h
```

---

## 使用多个命名空间

- 命名空间是在多个用户之间划分集群资源的一种方法（通过资源配额）。
  - 例如我们可以设置**开发**、**测试**、**生产**等多个命名空间。
- 不必使用多个命名空间来分隔轻微不同的资源。
  - 例如同一软件的不同版本： 应该使用标签 来区分同一命名空间中的不同资源。
- 命名空间适用于跨多个团队或项目的场景。
  - 对于只有几到几十个用户的集群，可以不用创建命名空间。
- 命名空间不能相互嵌套，每个 Kubernetes 资源只能在一个命名空间中。

---

## 管理命名空间

```sh
#创建命名空间
$ kubectl create ns develop
namespace/develop created

#在命名空间内运行Pod --namespace 等价于 -n
# kubectl run my-nginx --image=nginx -n=dev
$ kubectl run nginx --image=nginx:1.22 --namespace=develop
pod/nginx created

#查看命名空间内的Pod
$ kubectl get pod -n=develop
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          37s

# 默认是 default 下的命名空间
$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-5964889c54-fkd8t   1/1     Running   0          161m
nginx-deploy-5964889c54-lnd79   1/1     Running   0          161m
nginx-deploy-5964889c54-tfdf7   1/1     Running   0          161m
```

## 切换当前命名空间

```sh
$ kubectl config set-context $(kubectl config current-context) --namespace=develop
# 默认命名空间就改变为 develop 了
$ kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          3m25s
```

## 删除命名空间

```sh
$ kubectl delete ns develop
namespace "develop" deleted
# 因为默认被改为 develop 但现在develop 被删除了
$ kubectl get pod
No resources found in develop namespace.
# 查看ns状态
$ kubectl get ns
NAME              STATUS   AGE
default           Active   4d15h
kube-system       Active   4d15h
kube-public       Active   4d15h
kube-node-lease   Active   4d15h
# 切换为default
$ kubectl config set-context $(kubectl config current-context) --namespace=default
$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-5964889c54-fkd8t   1/1     Running   0          6h6m
nginx-deploy-5964889c54-lnd79   1/1     Running   0          6h6m
nginx-deploy-5964889c54-tfdf7   1/1     Running   0          6h6m
```
