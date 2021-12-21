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

  > 主人已实现kube-admin部署方式，[文档参考](https://github.com/ShouZhiDuan/deploy-work/blob/main/kubernetes/%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2/KubeAdmin%E9%83%A8%E7%BD%B2%E6%96%B9%E5%BC%8F/kubeadmin%E9%83%A8%E7%BD%B2%E6%96%B9%E5%BC%8F.md)。

- [kubespray方式](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubespray/)（官网）

  > 主人已实现kube-spray部署方式，部署文档待编写。

- [kops方式](https://kubernetes.io/zh/docs/setup/production-environment/tools/kops/)（官网）

  > 主人未实现。

- [hard-way方式](https://github.com/kelseyhightower)（社区）

  > 主人已实现hard-way部署方式，[文档参考](https://github.com/ShouZhiDuan/deploy-work/blob/main/kubernetes/%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2/%E4%BA%8C%E8%BF%9B%E5%88%B6%E9%83%A8%E7%BD%B2%E6%96%B9%E5%BC%8F/%E4%BA%8C%E8%BF%9B%E5%88%B6%E9%83%A8%E7%BD%B2%E6%95%99%E7%A8%8B.md)。

### 2、[k8s中文网](https://kubernetes.io/zh/docs/home/)

### 3、[在线服务器](https://labs.play-with-k8s.com/)

> 注意：这个有效时间4个小时，仅仅用于测试学习使用。

## 三、k8s初体验

> 1、定义一个pod_nginx_rs.yaml文件，熟悉docker-compose的话这里就不用解释了。

```powershell
cat > pod_nginx_rs.yaml <<EOF
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
  labels:
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: nginx
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
EOF
```

> 2、启动yaml

```powershell
kubectl apply -f pod_nginx_rs.yaml
```

> 3、查看pod

```powershell
kubectl get pods
kubectl get pods -o wide
kubectl describe pod nginx
```

> 4、扩容

```powershell
kubectl scale rs nginx --replicas=5
kubectl get pods -o wide
```

> 5、删除pod

```powershell
kubectl delete -f pod_nginx_rs.yaml
```

> 6、查看pod详情

```powershell
kubectl describe pod nginx-abc
```

```tex
Name:         nginx-abc-mhtjj
Namespace:    default
Priority:     0
Node:         node-3/192.168.10.123
Start Time:   Tue, 21 Dec 2021 16:59:20 +0800
Labels:       tier=frontend
Annotations:  cni.projectcalico.org/containerID: 8802143744ad0adbaa1bf2b0e8a63e4a4ef2379d45da808dbf962312c03d08b2
              cni.projectcalico.org/podIP: 10.200.139.106/32
              cni.projectcalico.org/podIPs: 10.200.139.106/32
Status:       Running
IP:           10.200.139.106
IPs:
  IP:           10.200.139.106
Controlled By:  ReplicaSet/nginx-abc
Containers:
  nginx-cname:
    Container ID:   containerd://fca76fd7a4d54e70e45eeb18b098a6f65611afc24e9de4e97c54610c3ad3f3b7
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:af472ddb9a3f053b8559361f89cfb7c2eb49775acff64b0999c08446f7e79b82
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 21 Dec 2021 16:59:23 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-44qj2 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-44qj2:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-44qj2
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:          <none>
```



















