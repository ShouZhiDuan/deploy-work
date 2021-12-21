# Kubernetes学习篇章

`作者：shouzhi@duan`

## 一、k8s核心组件

![image-20211221101609688](C:\Users\dev\AppData\Roaming\Typora\typora-user-images\image-20211221101609688.png)

### 1、Kubectl

> 用于在master节点操控集群节点工具，比如说创建pod、svc、configmap等相关资源。

### 2、ApiServer

> 请求到达master之后，然后在分配给worker工作节点创建Pod之类的关键命令是通过kubectl过来之后，是不是需要授权一下，那么ApiServer就是用于授权的组件。

### 3、Scheduler

> 授权通过后，接下来具体操作那个worker节点，或者container之类的，得要有调度策略。那么scheduler就是起到一个调度策略的作用。

### 4、Controller Manager

> 调度器执行调度之后会有一个分发器，用来真正路由分发到哪个worker工作节点上。

### 5、Kubelet

> 分发器分发到具体的worker后，最终会由kubelet服务调用Docker Engine，创建对应的容器。

### 6、DNS域名解析

> Calico、CoreDNS插件

### 7、ETCD分布式存储

> etcd集群部署。

### 8、面板监控

> Dashboard。

### 9、网络持、持久化

> 可以参考一下Docker方式，后面具体再做展开。

## 二、技术栈

### 1、k8s高可用部署方案

- [kubeadmin方式](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/)（官网）

  > 主人已实现kube-admin部署方式，部署文档待编写。

- [kubespray方式](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubespray/)（官网）

  > 主人已实现kube-spray部署方式，部署文档待编写。

- [kops方式](https://kubernetes.io/zh/docs/setup/production-environment/tools/kops/)（官网）

  > 主人未实现。

- [hard-way方式](https://github.com/kelseyhightower)（社区）

  > 主人已实现hard-way部署方式，[文档参考](https://github.com/ShouZhiDuan/deploy-work/blob/main/kubernetes/%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2/%E4%BA%8C%E8%BF%9B%E5%88%B6%E9%83%A8%E7%BD%B2%E6%96%B9%E5%BC%8F/%E4%BA%8C%E8%BF%9B%E5%88%B6%E9%83%A8%E7%BD%B2%E6%95%99%E7%A8%8B.md)。

### 2、[k8s中文网](https://kubernetes.io/zh/docs/home/)

### 3、[在线服务器](https://labs.play-with-k8s.com/)

> 注意：这个有效时间4个小时，仅仅用于测试学习使用。





































