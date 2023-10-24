# 声明式对象配置

<p class="r"> 
云原生的代表技术包括：
<ul>
  <li>容器</li>
  <li>服务网格</li>
  <li>微服务</li>
  <li>不可变基础设施</li>
  <li><label style="color:red">声明式API</label></li>
</ul>
</p>

## 管理对象

- 命令行指令
  例如，使用 kubectl 命令来创建和管理 Kubernetes 对象。
  命令行就好比口头传达，简单、快速、高效。
  但它功能有限，不适合复杂场景，操作不容易追溯，多用于开发和调试。
- 声明式配置
  kubernetes 使用 yaml 文件来描述 Kubernetes 对象。
  声明式配置就好比申请表，学习难度大且配置麻烦。
  好处是操作留痕，适合操作复杂的对象，多用于生产。

---

## 常用命令缩写

| 名称         | 缩写   | Kind        |
| ------------ | ------ | ----------- |
| namespaces   | ns     | Namespace   |
| nodes        | no     | Node        |
| pods         | po     | Pod         |
| services     | srv    | Service     |
| deployments  | deploy | Deployment  |
| replicasets  | rs     | ReplicaSet  |
| statefulsets | sts    | StatefulSet |

## YAML 规范

- 缩进代表上下级关系
- $\color{red} {缩进时不允许使用 Tab 键，只允许使用空格，通常缩进 2 个空格}$
- : 键值对，后面必须有空格
- -列表，后面必须有空格
- [ ]数组
- #注释
- | 多行文本块
- ---表示文档的开始，多用于分割多个资源对象

@import "./source/demo.yml"

---

## 配置对象

在创建的 Kubernetes 对象所对应的 yaml 文件中，需要配置的字段如下：

- `apiVersion` - Kubernetes API 的版本
- `kind` - 对象类别，例如 Pod、Deployment、Service、ReplicaSet 等
- `metadata` - 描述对象的元数据，包括一个 name 字符串、UID 和可选的 namespace
- `spec` - 对象的配置

---

<p class="r"> 
掌握程度：
<ul>
  <li>不要求自己会写</li>
  <li>找模版</li>
  <li>能看懂</li>
  <li>会修改</li>
  <li>能排错</li>
</ul>
</p>

---

使用 yaml 定义一个`Pod`
[Pod 配置模版](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#using-pods)

@import "source/my-pod.yaml"

```sh
# 执行
$ kubectl apply -f my-pod.yaml
pod/nginx created
# 查看 nginx 创建了
$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-5964889c54-fkd8t   1/1     Running   0          6h7m
nginx-deploy-5964889c54-lnd79   1/1     Running   0          6h7m
nginx-deploy-5964889c54-tfdf7   1/1     Running   0          6h7m
nginx                           1/1     Running   0          16s
# 删除
$ kubectl delete -f my-pod.yaml
pod "nginx" deleted
```

## 标签

**标签（Labels）** 是附加到对象（比如 Pod）上的键值对，用于补充对象的描述信息。
标签使用户能够以松散的方式管理对象映射，而无需客户端存储这些映射。
由于一个集群中可能管理成千上万个容器，我们可以使用标签高效的进行选择和操作容器集合。

---

- 键的格式：
  - $\color{red}{前缀}$(可选)/$\color{red}{名称}$(必须)。
- 有效名称和值：
  - 必须为 63 个字符或更少（可以为空）
  - 如果不为空，必须以字母数字字符（[a-z0-9A-Z]）开头和结尾
  - 包含破折号-、下划线\_、点.和字母或数字

[label 配置模版](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set)

@import "source/label-pod.yaml"

```sh
$ kubectl apply -f label-pod.yaml
pod/label-demo created
$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-5964889c54-fkd8t   1/1     Running   0          6h14m
nginx-deploy-5964889c54-lnd79   1/1     Running   0          6h14m
nginx-deploy-5964889c54-tfdf7   1/1     Running   0          6h14m
label-demo                      1/1     Running   0          5s
# 显示标签
$ kubectl get pod --show-labels
NAME                            READY   STATUS    RESTARTS   AGE     LABELS
nginx-deploy-5964889c54-fkd8t   1/1     Running   0          6h14m   app=nginx-deploy,pod-template-hash=5964889c54
nginx-deploy-5964889c54-lnd79   1/1     Running   0          6h14m   app=nginx-deploy,pod-template-hash=5964889c54
nginx-deploy-5964889c54-tfdf7   1/1     Running   0          6h14m   app=nginx-deploy,pod-template-hash=5964889c54
label-demo                      1/1     Running   0          11s     app=nginx,environment=production
# -l 过滤
$ kubectl get pod -l "app=nginx"
NAME         READY   STATUS    RESTARTS   AGE
label-demo   1/1     Running   0          39s
# , 分隔多个条件
$ kubectl get pod -l "app=nginx,environment=production"
NAME         READY   STATUS    RESTARTS   AGE
label-demo   1/1     Running   0          57s

```

---

## 选择器

**标签选择器** 可以识别一组对象。标签不支持唯一性。
标签选择器最常见的用法是为 Service 选择一组 Pod 作为后端。
[Service 配置模版](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#type-nodeport)

@import "source/my-service.yaml"

```sh
kubectl apply -f my-service.yaml
service/my-service created
$ kubectl get service
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP   10.43.0.1       <none>        443/TCP          4d18h
nginx-service   ClusterIP   10.43.146.225   <none>        8080/TCP         6h9m
nginx-outside   NodePort    10.43.4.121     <none>        8081:32555/TCP   5h50m
my-service      NodePort    10.43.134.176   <none>        80:30007/TCP     4s
$ kubectl describe svc/my-service
Name:                     my-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.134.176
IPs:                      10.43.134.176
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30007/TCP
Endpoints:                10.42.1.13:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
# 查看pod 是否被service 选中, 可以看到 Endpoints 对上
$ kubectl get pod -l "app=nginx" -owide
NAME         READY   STATUS    RESTARTS   AGE   IP           NODE               NOMINATED NODE   READINESS GATES
label-demo   1/1     Running   0          10m   10.42.1.13   k3d-demo-agent-1   <none>           <none>
```

目前支持两种类型的选择运算：**基于等值**的和**基于集合**的。
多个选择条件使用逗号分隔，相当于 **And(&&)**运算。

- **等值选择**

```yml
selector:
  matchLabels: # component=redis && version=7.0
    component: redis
    version: 7.0
```

- **集合选择**

```yml
selector:
  matchExpressions: # tier in (cache, backend) && environment not in (dev, prod)
    - { key: tier, operator: In, values: [cache, backend] }
    - { key: environment, operator: NotIn, values: [dev, prod] }
```

参考资料：  
https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/kubernetes-objects/
https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/object-management/
https://kubernetes.io/docs/reference/kubectl/#resource-types
https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/
https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/
