# Headless Service(无头服务)

之前我们创建了三个各自独立的数据库实例，mysql-0，mysql-1，mysql-2。
要想让别的容器访问数据库，我们需要将它发布为 Service，但是 Service 带负载均衡功能，每次请求都会转发给不同的数据库，这样子使用过程中会有很大的问题。

## 无头服务（Headless Services）

无头服务（Headless Service）可以为 StatefulSet 成员提供稳定的 DNS 地址。
在不需要负载均衡的情况下，可以通过指定 Cluster IP 的值为 "None" 来创建无头服务。

<p class="r">
<b>注意:</b><code>StatefulSet</code>中的<code>ServiceName</code>必须要跟Service中的<code>metadata.name</code>一致
</p>

@import "source/mysql/statefulset_headless.yaml"

```sh
$ kubectl apply -f statefulset_headless.yaml
service/mysql-svc created
statefulset.apps/mysql-sts configured
$ kubectl describe svc/mysql-svc
Name:              mysql-svc
Namespace:         default
Labels:            app=mysql-svc
Annotations:       <none>
Selector:          app=mysql
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                None
IPs:               None
Port:              mysql  3306/TCP
TargetPort:        3306/TCP
Endpoints:         10.42.0.6:3306,10.42.1.8:3306,10.42.2.12:3306
Session Affinity:  None
Events:            <none>
# 可以看到 CLUSTER-IP 为 None
$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP    177m
mysql-svc    ClusterIP   None         <none>        3306/TCP   37s
```

### 稳定的网络 ID

StatefulSet 中的每个 Pod 都会被分配一个 StatefulSet 名称-序号格式的主机名。
集群内置的 DNS 会为 Service 分配一个内部域名 db.default.svc.cluster.local,它的格式为 服务名称.命名空间.svc.cluster.local。
Service 下的每个 Pod 会被分配一个子域名，格式为 pod 名称.所属服务的域名，例如 mysql-0 的域名为 mysql-0.db.default.svc.cluster.local。
创建 Pod 时，DNS 域名生效可能会有一些延迟(几秒或几十秒)。
Pod 之间可以通过 DNS 域名访问，同一个命名空间下可以省略命名空间及其之后的内容。

- 查看主节点 host 文件

```sh
$ cat /etc/hosts
# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
10.42.1.8       mysql-sts-0.mysql-svc.default.svc.cluster.local mysql-sts-0
```

- 新 pod 可以看到访问路由

```sh
$ kubectl run dns-test -ti --image=busybox:1.28 --rm
If you don't see a command prompt, try pressing enter.
/ # nslookup mysql-svc
Server:    10.43.0.10
Address 1: 10.43.0.10 kube-dns.kube-system.svc.cluster.local

Name:      mysql-svc
Address 1: 10.42.0.6 mysql-sts-2.mysql-svc.default.svc.cluster.local
Address 2: 10.42.1.8 mysql-sts-0.mysql-svc.default.svc.cluster.local
Address 3: 10.42.2.12 mysql-sts-1.mysql-svc.default.svc.cluster.local
/ # nslookup mysql-sts-0.mysql-svc
Server:    10.43.0.10
Address 1: 10.43.0.10 kube-dns.kube-system.svc.cluster.local

Name:      mysql-sts-0.mysql-svc
Address 1: 10.42.1.8 mysql-sts-0.mysql-svc.default.svc.cluster.local
```

---

参考文档：
https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/
https://kubernetes.io/zh-cn/docs/tutorials/stateful-application/basic-stateful-set/
https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#headless-services
https://kubernetes.io/zh-cn/docs/tasks/run-application/run-replicated-stateful-application/
https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/
