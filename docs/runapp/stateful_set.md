# StatefulSet(有状态应用集)

## StatefulSet

如果我们需要部署多个 MySQL 实例，就需要用到 StatefulSet。
StatefulSet 是用来管理有状态的应用。一般用于管理数据库、缓存等。
与 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 类似， StatefulSet 用来管理 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 集合的部署和扩缩。
Deployment 用来部署无状态应用。StatefulSet 用来有状态应用。

---

## 创建 StatefulSet

[StatefulSet 配置模版](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#components)

@import "source/mysql/statefulset.yaml"

### 稳定的存储

在 StatefulSet 中使用 VolumeClaimTemplate，为每个 Pod 创建持久卷声明(PVC)。
每个 Pod 将会得到基于 local-path 存储类动态创建的持久卷(PV)。 Pod 创建(或重新调度）时，会挂载与其声明相关联的持久卷。
请注意，当 Pod 或者 StatefulSet 被删除时，持久卷声明和关联的持久卷不会被删除。

---

### Pod 标识

在具有 N 个副本的 StatefulSet 中，每个 Pod 会被分配一个从 0 到 N-1 的整数序号，该序号在此 StatefulSet 上是唯一的。
StatefulSet 中的每个 Pod 主机名的格式为 `StatefulSet 名称-序号`。
上例将会创建三个名称分别为 mysql-0、mysql-1、mysql-2 的 Pod。

---

## 部署和扩缩保证

- 对于包含 N 个 副本的 StatefulSet，当部署 Pod 时，它们是依次创建的，顺序为 0..N-1。
- 当删除 Pod 时，它们是逆序终止的，顺序为 N-1..0。
- 在将扩缩操作应用到 Pod 之前，它前面的所有 Pod 必须是 Running 和 Ready 状态。
- 在一个 Pod 终止之前，所有的继任者必须完全关闭。

---

```sh
# 运行
$ kubectl apply -f statefulset.yaml
statefulset.apps/mysql-sts created
# 观察进度
$ kubectl get pod --watch
NAME          READY   STATUS    RESTARTS   AGE
mysql-sts-0   0/1     Pending   0          0s
mysql-sts-0   0/1     Pending   0          17s
mysql-sts-0   0/1     ContainerCreating   0          17s
mysql-sts-0   1/1     Running             0          22s
mysql-sts-1   0/1     Pending             0          0s
mysql-sts-1   0/1     Pending             0          18s
mysql-sts-1   0/1     ContainerCreating   0          19s
mysql-sts-1   1/1     Running             0          35s
mysql-sts-2   0/1     Pending             0          0s
mysql-sts-2   0/1     Pending             0          16s
mysql-sts-2   0/1     ContainerCreating   0          16s
mysql-sts-2   1/1     Running             0          63s
```

在上面的 mysql 示例被创建后，会按照 mysql-0、mysql-1、mysql-2 的顺序部署三个 Pod。
在 mysql-0 进入 [Running 和 Ready](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/) 状态前不会部署 mysql-1。
在 mysql-1 进入 Running 和 Ready 状态前不会部署 mysql-2。
如果 mysql-1 已经处于 Running 和 Ready 状态，而 mysql-2 尚未部署，在此期间发生了 mysql-0 运行失败，那么 mysql-2 将不会被部署，要等到 mysql-0 部署完成并进入 Running 和 Ready 状态后，才会部署 mysql-2。
如果用户想将示例中的 StatefulSet 扩缩为 replicas=1，首先被终止的是 mysql-2。
在 mysql-2 没有被完全停止和删除前，mysql-1 不会被终止。
当 mysql-2 已被终止和删除、mysql-1 尚未被终止，如果在此期间发生 mysql-0 运行失败， 那么就不会终止 mysql-1，必须等到 mysql-0 进入 Running 和 Ready 状态后才会终止 web-1。

---

参考文档：  
https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/
https://kubernetes.io/zh-cn/docs/tasks/run-application/run-replicated-stateful-application/
