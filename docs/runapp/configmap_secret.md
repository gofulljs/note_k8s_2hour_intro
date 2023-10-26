# ConfigMap 与 Secret

<p class="r">
在Docker中，我们一般通过绑定挂载的方式将配置文件挂载到容器里。
在Kubernetes集群中，容器可能被调度到任意节点，配置文件需要能在集群任意节点上访问、分发和更新。
</p>

## ConfigMap

ConfigMap 用来在键值对数据库(etcd)中保存非加密数据。一般用来保存配置文件。
ConfigMap 可以用作环境变量、命令行参数或者存储卷。
ConfigMap 将环境配置信息与 [容器镜像](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-image) 解耦，便于配置的修改。
ConfigMap 在设计上不是用来保存大量数据的。
在 ConfigMap 中保存的数据不可超过 1 MiB。
超出此限制，需要考虑挂载存储卷或者访问文件存储服务。

---

### ConfigMap 用法

- [ConfigMap 配置示例](https://kubernetes.io/docs/concepts/configuration/configmap/#configmaps-and-pods)
- [Pod 中使用 ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/#using-configmaps-as-files-from-a-pod)

@import "source/mysql/mysql-pod_configmap.yml"

```sh
# 获取对应键值信息， configMap 也可缩写为 cm
$ kubectl describe configMap mysql-config
Name:         mysql-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
mysql.cnf:
----
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci
init-connect='SET NAMES utf8mb4'

[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4


BinaryData
====

Events:  <none>

# 获取信息
$ kubectl get pod -owide
NAME        READY   STATUS    RESTARTS   AGE   IP          NODE               NOMINATED NODE   READINESS GATES
mysql-pod   1/1     Running   0          2m    10.42.1.5   k3d-demo-agent-1   <none>           <none>

# 进入mysql 查看字符集
$ kubectl exec -ti mysql-pod -- bash
bash-4.2# mysql -uroot -p123456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.43 MySQL Community Server (GPL)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show variables like "%char%"
    -> ;
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

- 修改配置文件

```sh
$ kubectl edit cm mysql-config
```

在`[client]`上分增加一行注释 `# this is a new comment`, 查看配置文件已修改：

```sh
$ kubectl exec -ti mysql-pod -- bash
bash-4.2# cat /etc/mysql/conf.d/mysql.cnf
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci
init-connect='SET NAMES utf8mb4'
# this is a new comment
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4
```

## Secret

Secret 用于保存机密数据的对象。一般由于保存密码、令牌或密钥等。
`data` 字段用来存储 base64 编码数据。
`stringData` 存储未编码的字符串。
Secret 意味着你不需要在应用程序代码中包含机密数据，减少机密数据(如密码)泄露的风险。
Secret 可以用作环境变量、命令行参数或者存储卷文件。

---

Secret 用法

- [Secret 配置示例](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/#use-case)
- [将 Secret 用作环境变量](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

```sh
$ echo -n '123456' | base64
MTIzNDU2
$ echo 'MTIzNDU2' | base64 --decode
123456
```

@import "source/mysql/mysql-pod_secret.yml"

- 查看 secret

```sh
$ kubectl apply -f mysql-pod_secret.yml
secret/mysql-password created
pod/mysql-pod created
configmap/mysql-config configured
$ kubectl describe secret/mysql-password
Name:         mysql-password
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
PASSWORD:  7 bytes
```
