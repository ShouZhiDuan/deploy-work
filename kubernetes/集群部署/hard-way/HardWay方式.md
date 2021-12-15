# Kubernetes二进制部署教程

## 一、部署特点

- 二进制部署需要大家手动部署，下载集群中所有相关部署组件，从而让大家深入了解各组件的角色功能。

- 生产级别的高可用部署方式，避免其他工具部署方式（kube-admin、kubespray）带来相关问题。

  - 部署过程中个组件版本兼容问题。
  - 部署过程中组件下载超时或者失败的问题。
  - 部署过程中工具脚本(ansible)自动化错误问题，难以介入解决。
  - 证书过期问题等。

- 高可用不依赖haproxy、keepalived。采用本地代理方式，简单优雅。

  

## 二、适合群体

- 对kubernetes系统有一定的基础认知。

- 想深入学习kubernetes。
- 对kubernetes二进制部署有强烈的兴趣。
- 正在接触或者学习部署生产级别kubernetes高可用集群。



## 三、部署文档

### 1-基础环境准备

#### 1.1服务器说明

##### 1.1.1 节点要求

- 节点数>=3台
- CPU>=2
- 内存>=2G
- 关闭安全组，允许节点之间任意端口访问，以及ipip隧道协议通讯

##### 1.1.2 服务器分配说明

| 系统类型 | IP             | 角色           | CPU  | 内存/G | HostName |
| -------- | -------------- | -------------- | ---- | ------ | -------- |
| Ubuntu   | 192.168.10.121 | master         | 8    | 15     | fabric1  |
| Ubuntu   | 192.168.10.122 | master\|worker | 8    | 15     | fabric1  |
| Ubuntu   | 192.168.10.123 | worker         | 8    | 15     | fabric1  |

#### 1.2系统设置（所有集群机器都要操作）

##### 1.2.1域名映射

```tex
vi /etc/hosts
#kubernetes
192.168.10.121 node-1
192.168.10.122 node-2
192.168.10.123 node-3
```

##### 1.2.2下载相关软件包

```tex
socat conntrack ipvsadm ipset jq sysstat curl iptables libseccomp yum-utils
```

##### 1.2.3关闭防火墙、selinux、swap，重置iptables

```powershell
# 关闭selinux
$ setenforce 0
$ sed -i '/SELINUX/s/enforcing/disabled/' /etc/selinux/config
# 关闭防火墙
$ systemctl stop firewalld && systemctl disable firewalld

# 设置iptables规则
$ iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT
# 关闭swap
$ swapoff -a && free –h

# 关闭dnsmasq(否则可能导致容器无法解析域名)
$ service dnsmasq stop && systemctl disable dnsmasq
```

##### 1.2.3kubernetes参数配置

```tex
# 制作配置文件
$ cat > /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_forward = 1
vm.swappiness = 0
vm.overcommit_memory = 1
EOF
# 生效文件
$ sysctl -p /etc/sysctl.d/kubernetes.conf
```

##### 1.2.4免密登录配置

```tex
# 看看是否已经存在rsa公钥
$ cat ~/.ssh/id_rsa.pub

# 如果不存在就创建一个新的
$ ssh-keygen -t rsa

# 把id_rsa.pub文件内容copy到其他机器的授权文件中
$ cat ~/.ssh/id_rsa.pub

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGVY93IOuYzT7C1Z6ZsyqNjDYMzF9QsiRE2rYVjtN+yQ18zVEmwPGVvHrJgqx7TDd2ir2cfMi9whpADA65L/LHDub2PK1OSB5OMdS2gaMFoSkoCtzlz+nkkEH0YGIRBbJUJ944Ha8MWSwWEHd0K/7F+F1Y2DpPMfRT4Ohaond2oKYnDA0r8LnOOJSMdMprGBnVtRdSR+8fxgJadGhbJReLjyJRdrMzW1cvJUXfp2DeR68jS7fxOd2vEV8+8S679aJvIwc+3X51WNYaKHx0I4fRMMvFusIFPZxD6G9h6Lm+mzVpFIgFfopfcyQ3QRO4sqSKexbRoCHm8YXNlq3RuKbB root@fabric1

# 在其他节点执行下面命令（包括worker节点）
$ echo "<file_content>" >> ~/.ssh/authorized_keys


echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGVY93IOuYzT7C1Z6ZsyqNjDYMzF9QsiRE2rYVjtN+yQ18zVEmwPGVvHrJgqx7TDd2ir2cfMi9whpADA65L/LHDub2PK1OSB5OMdS2gaMFoSkoCtzlz+nkkEH0YGIRBbJUJ944Ha8MWSwWEHd0K/7F+F1Y2DpPMfRT4Ohaond2oKYnDA0r8LnOOJSMdMprGBnVtRdSR+8fxgJadGhbJReLjyJRdrMzW1cvJUXfp2DeR68jS7fxOd2vEV8+8S679aJvIwc+3X51WNYaKHx0I4fRMMvFusIFPZxD6G9h6Lm+mzVpFIgFfopfcyQ3QRO4sqSKexbRoCHm8YXNlq3RuKbB root@fabric1" >> ~/.ssh/authorized_keys
```

#### 1.3准备kubernetes软件包

> 6个软件包

```tex
master节点组件：kube-apiserver、kube-controller-manager、kube-scheduler、kubectl
worker节点组件：kubelet、kube-proxy
```

> 下载教程（需要梯子）

```powershell
# 设定版本号
$ export VERSION=v1.20.2

# 下载master节点组件
$ wget https://storage.googleapis.com/kubernetes-release/release/${VERSION}/bin/linux/amd64/kube-apiserver
$ wget https://storage.googleapis.com/kubernetes-release/release/${VERSION}/bin/linux/amd64/kube-controller-manager
$ wget https://storage.googleapis.com/kubernetes-release/release/${VERSION}/bin/linux/amd64/kube-scheduler
$ wget https://storage.googleapis.com/kubernetes-release/release/${VERSION}/bin/linux/amd64/kubectl

# 下载worker节点组件
$ wget https://storage.googleapis.com/kubernetes-release/release/${VERSION}/bin/linux/amd64/kube-proxy
$ wget https://storage.googleapis.com/kubernetes-release/release/${VERSION}/bin/linux/amd64/kubelet

# 下载etcd组件
$ wget https://github.com/etcd-io/etcd/releases/download/v3.4.10/etcd-v3.4.10-linux-amd64.tar.gz
$ tar -xvf etcd-v3.4.10-linux-amd64.tar.gz
$ mv etcd-v3.4.10-linux-amd64/etcd* .
$ rm -fr etcd-v3.4.10-linux-amd64*

# 统一修改文件权限为可执行
$ chmod +x kube*
```

> 如果没有梯子下载，可以[点击下载](https://pan.baidu.com/s/1FkLI2WU8oo6wEUWLVPafwQ)，这里面包含ETCD以及以上6个软件包
>
> 提取码：`kwvv`

#### 1.4软件包分发

完成下载后，将这个 6个软件包分发到各个角色主机相关目录。不同的角色主机需要的相关软件包会有不同。

```powershell
# 把master相关组件分发到master节点
$ MASTERS=(node-1 node-2)
for instance in ${MASTERS[@]}; do
  scp kube-apiserver kube-controller-manager kube-scheduler kubectl root@${instance}:/usr/local/bin/
done

# 把worker先关组件分发到worker节点
$ WORKERS=(node-2 node-3)
for instance in ${WORKERS[@]}; do
  scp kubelet kube-proxy root@${instance}:/usr/local/bin/
done

# 把etcd组件分发到etcd节点
$ ETCDS=(node-1 node-2 node-3)
for instance in ${ETCDS[@]}; do
  scp etcd etcdctl root@${instance}:/usr/local/bin/
done
```







