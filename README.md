# k8s 课程导读

a note use gitbook for Kubernetes 二小时入门教程,

根据自己的学习会做一些添加和补充

## B 站视频地址

https://www.bilibili.com/video/BV1k24y197KC

## 原文档地址

https://www.yuque.com/wukong-zorrm/qdoy5p

## 为什么 Kubernetes 学起来很难？

zh

- Kubernetes 本身比较复杂，组件众多，安装过程比较麻烦

  - 本课程使用 K3s 快速创建学习环境，不要把时间和精力浪费在搭环境上

- 网络问题，许多谷歌镜像或软件仓库访问不到，拉取失败

  - 配置阿里云镜像加速
  - 手动拉取镜像、手动导出、导入镜像

- Kubernetes 版本有重大变化，网上好多教程已过时

  - kubernetes 从 1.24 版本开始，移除了对 docker 的支持
  - 本课程采用 1.25 版本，使用 containerd 作为容器运行时
  - 课程中对 containerd 用法以及可能遇到的问题进行了说明

- 官方文档有错误，许多例子或命令运行不起来

  - 本课程会帮你避过官方文档中的坑

- 很多教程只有例子，没有实战，导致“一学就会，一用就废”

  - 本课程会演示常用中间件的安装（MySQL 主从集群、Redis 主从集群）
  - 本课程会演示如何在 K8s 上运行一个完整的应用
    - 应用程序包括前端(node/nginx)、缓存(redis)、数据库(mysql)、后端(java）
