# 安装 Minikube

docker-desktop 新版本自带了 kubenetes, 不需要使用 minikube 了。

自带的安装较慢可以使用 https://github.com/AliyunContainerService/k8s-for-docker-desktop 去安装 kubenetes 和 dashboard

## dashbord

https://github.com/kubernetes/dashboard

- 安装官方版

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

查看是否允许成功

```sh
$ kubectl get pods -A
NAMESPACE              NAME                                         READY   STATUS    RESTARTS        AGE
kube-system            coredns-5d78c9869d-tjhgv                     1/1     Running   5 (7m45s ago)   2d22h
kube-system            coredns-5d78c9869d-vbzjn                     1/1     Running   5 (7m45s ago)   2d22h
kube-system            etcd-docker-desktop                          1/1     Running   5 (7m49s ago)   2d22h
kube-system            kube-apiserver-docker-desktop                1/1     Running   5 (7m40s ago)   2d22h
kube-system            kube-controller-manager-docker-desktop       1/1     Running   5 (7m50s ago)   2d22h
kube-system            kube-proxy-nrmkx                             1/1     Running   5 (7m50s ago)   2d22h
kube-system            kube-scheduler-docker-desktop                1/1     Running   5 (7m50s ago)   2d22h
kube-system            storage-provisioner                          1/1     Running   9 (6m28s ago)   2d22h
kube-system            vpnkit-controller                            1/1     Running   5 (7m50s ago)   2d22h
kubernetes-dashboard   dashboard-metrics-scraper-5cb4f4bb9c-ws8sk   1/1     Running   1 (7m50s ago)   11m
kubernetes-dashboard   kubernetes-dashboard-6967859bff-5tvh6        1/1     Running   1 (7m50s ago)   11m
```

以上后 2 行出现代表成功

启用 dashboard

```sh
kubectl proxy
```

kubectl 会使得 Dashboard 可以通过 http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/ 访问。

UI 只能 通过执行这条命令的机器进行访问。

- windows 环境获取 Token

```powershell
$TOKEN=((kubectl -n kube-system describe secret default | Select-String "token:") -split " +")[1]
kubectl config set-credentials docker-desktop --token="${TOKEN}"
echo $TOKEN
```

参考：

https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/web-ui-dashboard/
