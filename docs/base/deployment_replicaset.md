# Deployment(部署)与 ReplicaSet(副本集)

**Deployment**是对 ReplicaSet 和 Pod 更高级的抽象。
它使 Pod 拥有多副本，自愈，扩缩容、滚动升级等能力。

**ReplicaSet**(副本集)是一个 Pod 的集合。
它可以设置运行 Pod 的数量，确保任何时间都有指定数量的 Pod 副本在运行。
通常我们不直接使用 ReplicaSet，而是在 Deployment 中声明。

```sh
#创建deployment,部署3个运行nginx的Pod
$ kubectl create deployment nginx-deploy --image=nginx:1.22 --replicas=3
#查看deployment(缩写成deploy)
$ kubectl get deploy
# 查看pod
$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-5964889c54-zmk6p   1/1     Running   0          6m1s
nginx-deploy-5964889c54-th9f8   1/1     Running   0          6m1s
nginx-deploy-5964889c54-28689   1/1     Running   0          6m1s
#查看replicaSet(缩写为rs)
$ kubectl get replicaSet
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-5964889c54   3         3         3       7m13s
```

可以看到 5964889c54 为副本集 ， 有 3 个 pod 分别为 zmk6p、th9f8、28689

- 删除最后一个 pod 会再生成新的 pod

```sh
$ kubectl delete pod nginx-deploy-5964889c54-28689
pod "nginx-deploy-5964889c54-28689" deleted
# 可以看到生成了新的pod k9xnj，体现了 deploy 使pod拥有的自愈能力
$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-5964889c54-zmk6p   1/1     Running   0          9m29s
nginx-deploy-5964889c54-th9f8   1/1     Running   0          9m29s
nginx-deploy-5964889c54-k9xnj   1/1     Running   0          13s
```

## 缩放

- 手动缩放

```sh
# 副本扩容到5个
$ kubectl scale deploy nginx-deploy --replicas=5
```

扩容过程

```sh
# DESIRED 目标  CURRENT 当前   READY 准备就绪
$ kubectl get rs --watch
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-5964889c54   3         3         3       12m
nginx-deploy-5964889c54   5         3         3       13m // Step A
nginx-deploy-5964889c54   5         3         3       13m
nginx-deploy-5964889c54   5         4         3       13m // Step B
nginx-deploy-5964889c54   5         5         3       13m // Step C
nginx-deploy-5964889c54   5         5         4       13m // Step D
nginx-deploy-5964889c54   5         5         5       13m // Step E
```

可以看到在观测副本时过程

1. 执行了扩容后 DESIRED 就变成了 5 (Step A)
2. CURRENT 变成 4， 证明 启动了一个 pod，READY 还是 3， 说明还没启动完成 (Step B)
3. CURRENT 变成 5， 证明 又启动了一个 pod，READY 还是 3， 说明还没启动完成 (Step C)
4. READY 变成了 4， 说明成功运行了一个 pod (Step D)
5. READY 变成了 5， 说明又成功运行了一个 pod, 这样 READY 和 DESIRED 相等，扩容完成 (Step D)

调整回去

```sh
$ kubectl scale deploy nginx-deploy --replicas=3
```

过程

```sh
$ kubectl get rs --watch
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-5964889c54   5         5         5       27m
nginx-deploy-5964889c54   3         5         5       28m
nginx-deploy-5964889c54   3         5         5       28m
nginx-deploy-5964889c54   3         3         3       28m
```

- 自动缩放

自动缩放通过增加和减少副本的数量，以保持所有 Pod 的平均 CPU 利用率不超过 75%。
自动伸缩需要声明 Pod 的资源限制，同时使用 [Metrics Server 服务](https://github.com/kubernetes-sigs/metrics-server#readme)（K3s 默认已安装）。

<p class="r">本例仅用来说明kubectl autoscale命令的使用，完整示例参考：<a href="https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/">HPA演示</a><p>

```sh
#自动缩放
kubectl autoscale deployment/nginx-auto --min=3 --max=10 --cpu-percent=75
#查看自动缩放
kubectl get hpa
#删除自动缩放
kubectl delete hpa nginx-deployment
```

## 滚动更新

- 设置镜像为 1.23

```sh
$ kubectl set image deploy/nginx-deploy nginx=nginx:1.23
# 查看滚动更新状态
$ kubectl rollout status deployment/nginx-deploy
deployment "nginx-deploy" successfully rolled out
```

- 查看更新后的版本和 pod

```sh
$ kubectl get deploy/nginx-deploy -owide
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES       SELECTOR
nginx-deploy   3/3     3            3           39m   nginx        nginx:1.23   app=nginx-deploy
```

- 过程及说明

```sh
$ kubectl get rs --watch
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-5964889c54   3         3         3       35m
nginx-deploy-7c88b8c7c9   1         0         0       0s  // Step 1
nginx-deploy-7c88b8c7c9   1         0         0       0s
nginx-deploy-7c88b8c7c9   1         1         0       0s  // Step 2
nginx-deploy-7c88b8c7c9   1         1         1       22s // Step 3
nginx-deploy-5964889c54   2         3         3       37m // Step 4
nginx-deploy-5964889c54   2         3         3       37m
nginx-deploy-5964889c54   2         2         2       37m // Step 5
nginx-deploy-7c88b8c7c9   2         1         1       23s // Step 6
nginx-deploy-7c88b8c7c9   2         1         1       23s
nginx-deploy-7c88b8c7c9   2         2         1       23s
nginx-deploy-7c88b8c7c9   2         2         2       43s // Step 7
nginx-deploy-5964889c54   1         2         2       37m
nginx-deploy-7c88b8c7c9   3         2         2       43s
nginx-deploy-5964889c54   1         2         2       37m
nginx-deploy-5964889c54   1         1         1       37m
nginx-deploy-7c88b8c7c9   3         2         2       43s
nginx-deploy-7c88b8c7c9   3         3         2       43s
nginx-deploy-7c88b8c7c9   3         3         3       63s
nginx-deploy-5964889c54   0         1         1       37m
nginx-deploy-5964889c54   0         1         1       37m
nginx-deploy-5964889c54   0         0         0       37m
```

可以看到 7c88b8c7c9 为新部署 nginx 的 deploy， 5964889c54 为 之前部署的 nginx 的 deploy

step:

1. 新 deploy 目标为 1 个 pod，当前和就绪为 0
2. 新 deploy 1 个正在 启动
3. 新 deploy 成功启动 1 个
4. 旧 deploy 准备减少一个
5. 旧 deploy 减少一个完成
6. 新 deploy 准备再起 1 个 pod
7. 新 deploy 成功又启动了 1 个 pod

后续步骤同理直到 新 deploy 满足要求 旧 deploy 停止所有 pod，副本集清 0

- 版本回滚

```sh
#查看历史版本
$ kubectl rollout history deployment/nginx-deploy
deployment.apps/nginx-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
# 查看版本1详情 5964889c54 为版本1 hash值
$ kubectl rollout history deployment/nginx-deploy --revision=1
deployment.apps/nginx-deploy with revision #1
Pod Template:
  Labels:       app=nginx-deploy
        pod-template-hash=5964889c54
  Containers:
   nginx:
    Image:      nginx:1.22
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
#回滚到历史版本
$ kubectl rollout undo deployment/nginx-deploy --to-revision=1
deployment.apps/nginx-deploy rolled back
```

回滚过程

```sh
$ kubectl get rs --watch
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-7c88b8c7c9   3         3         3       20m
nginx-deploy-5964889c54   0         0         0       57m
nginx-deploy-5964889c54   0         0         0       57m
nginx-deploy-5964889c54   1         0         0       57m
nginx-deploy-5964889c54   1         0         0       57m
nginx-deploy-5964889c54   1         1         0       57m
nginx-deploy-5964889c54   1         1         1       57m
nginx-deploy-7c88b8c7c9   2         3         3       21m
nginx-deploy-7c88b8c7c9   2         3         3       21m
nginx-deploy-5964889c54   2         1         1       57m
nginx-deploy-7c88b8c7c9   2         2         2       21m
nginx-deploy-5964889c54   2         1         1       57m
nginx-deploy-5964889c54   2         2         1       57m
nginx-deploy-5964889c54   2         2         2       57m
nginx-deploy-7c88b8c7c9   1         2         2       21m
nginx-deploy-5964889c54   3         2         2       57m
nginx-deploy-7c88b8c7c9   1         2         2       21m
nginx-deploy-7c88b8c7c9   1         1         1       21m
nginx-deploy-5964889c54   3         2         2       57m
nginx-deploy-5964889c54   3         3         2       57m
nginx-deploy-5964889c54   3         3         3       57m
nginx-deploy-7c88b8c7c9   0         1         1       21m
nginx-deploy-7c88b8c7c9   0         1         1       21m
nginx-deploy-7c88b8c7c9   0         0         0       21m
```

查看回滚后的状态

```sh
# 查看副本集状态
$ kubectl get rs
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-5964889c54   3         3         3       59m
nginx-deploy-7c88b8c7c9   0         0         0       22m
# 查看pod状态，发现副本集回滚了
$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-5964889c54-fkd8t   1/1     Running   0          2m31s
nginx-deploy-5964889c54-lnd79   1/1     Running   0          2m30s
nginx-deploy-5964889c54-tfdf7   1/1     Running   0          2m28s
```

删除无用副本集

```sh
$ kubectl delete rs nginx-deploy-7c88b8c7c9
replicaset.apps "nginx-deploy-7c88b8c7c9" deleted
```

参考文档：  
https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/
https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/
https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
