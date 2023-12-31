# 运行有状态应用

我们以 MySQL 数据库为例，在 kubernetes 集群中运行一个有状态的应用。
部署数据库几乎覆盖了 kubernetes 中常见的对象和概念：

- **配置文件--ConfigMap**
- **保存密码--Secret**
- **数据存储--持久卷(PV)和持久卷声明(PVC)**
- **动态创建卷--存储类(StorageClass)**
- **部署多个实例--StatefulSet**
- **数据库访问--Headless Service**
- **主从复制--初始化容器和 sidecar**
- **数据库调试--port-forward**
- **部署 Mysql 集群--helm**
