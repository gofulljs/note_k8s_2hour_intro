# 创建 MySQL 数据库

- 配置环境变量

  - 使用 [MySQL 镜像](https://hub.docker.com/_/mysql) 创建 Pod，需要使用环境变量设置 MySQL 的初始密码。
  - [环境变量配置示例](https://kubernetes.io/zh-cn/docs/tasks/inject-data-application/define-environment-variable-container/#define-an-env-variable-for-a-container)

- 挂载卷
  - 将数据存储在容器中，一旦容器被删除，数据也会被删除。
  - 将数据存储到卷(Volume)中，删除容器时，卷不会被删除。

---

## hostPath 卷

hostPath 卷将主机节点上的文件或目录挂载到 Pod 中。
[hostPath 配置示例](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath-configuration-example)

---

**hostPath 的 type 值：**

| DirectoryOrCreate | DirectoryOrCreate                                                                      |
| ----------------- | -------------------------------------------------------------------------------------- |
| **Directory**     | **挂载已存在目录。不存在会报错。**                                                     |
| **FileOrCreate**  | **文件不存在则自动创建。**<br>**不会自动创建文件的父目录，必须确保文件路径已经存在。** |
| **File**          | **挂载已存在的文件。不存在会报错。**                                                   |
| **Socket**        | **挂载 UNIX 套接字。例如挂载/var/run/docker.sock 进程**                                |

@import "source/mysql/mysql-pod.yml"

<p class="r">
<b>注意：hostPath</b> 仅用于在单节点集群上进行开发和测试，不适用于多节点集群；
例如，当Pod被重新创建时，可能会被调度到与原先不同的节点上，导致新的Pod没有数据。
在多节点集群使用本地存储，可以使用local卷。
</p>

---

参考文档：  
https://kubernetes.io/zh-cn/docs/tasks/inject-data-application/define-environment-variable-container/
https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#hostpath
https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
