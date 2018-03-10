---
title: 使用kubeadm创建k8s集群
date: 2018-03-10 13:34:21
tags:
categories:
---

{% note info %}

kubeadm是Kubernetes官方提供的快速安装和初始化Kubernetes集群的工具
大大简化了部署的难度，参照前人的经验，自己动手实现了一番
唯一的困难镜像在国外，你懂的

{% endnote %}

<!-- more -->

使用kubeadm创建k8s集群
---

### 1. 硬件情况
virtual Box 虚拟机两台
系统：centos7.4

| Role   | Nums |  配置  |       IP       |  主机名   |
| :----- | ---: | :--: | :------------: | :----: |
| master |    1 | 1核2G | 192.168.90.89  | master |
| node   |    1 | 1核2G | 192.168.90.228 | node1  |

写入`/etc/hosts`,两台
```
192.168.90.89 master
192.168.90.228 node1
```


#### 1.1配置代理
**参考:**
{% post_link 25-centos7Proxy-md %}


#### 1.2系统设置
1. 搭建测试环境,关闭防火墙
  + `systemctl stop firewalld`
  + `systemctm disable firewalld`

2. 关闭swap空间
  + `swapoff -a`	
  + 修改`/etc/fstab`,注释掉swap自动挂载

3. 关闭selinux
  + `vim /etc/sysconfig/selinux`,修改成`disabled`
  + `setenforce 0`

4. 配置yum源
```bash
# k8s
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

#yum源,可以使用aliyun的,也可以使用自带的centos.org
```

> **特别注意:**
> 如果你安装的是k8s 1.9,不要修改docker和kubelet的cgroup驱动,改了反而报错

#### 1.3调整内核参数
```bash
$ cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

#执行
sysctl --system
```
### 2. 安装
都需安装docker, kubeadm, kubelet和kubectl 
使用的下面版本:
```bash
kubeadm-1.9.2-0.x86_64
kubectl-1.9.2-0.x86_64
kubelet-1.9.2-0.x86_64
docker-1.12.6-71.git3e8e77d.el7.centos.1.x86_64
```

#### 2.1首先安装docker
```bash
yum install -y docker
systemctl enable docker 
systemctl start docker
```

因为很多镜像在google上面的,所以**docker也需要配置代理**
根据官网的文档:[连接](https://docs.docker.com/engine/admin/systemd/#httphttps-proxy)

1.为docker服务创建一个systemd的drop-in目录：
```bash
mkdir -p /etc/systemd/system/docker.service.d
```
2.创建一个名为`/etc/systemd/system/docker.service.d/http-proxy.conf`的文件，该文件添加了`http_proxy`环境变量：
```bash
[Service]
Environment="HTTPS_PROXY=https://127.0.0.1:8118/" "NO_PROXY=localhost,172.16.0.0/16,192.168.0.0/16,127.0.0.1,10.10.0.0/16,10.244.0.0/16"
```
3.刷新更改：
```bash
systemctl daemon-reload
```
4.重启Docker:
```bash
systemctl restart docker
```
5.验证结果是否加载:
```bash
$ systemctl show --property=Environment docker |more
Environment=GOTRACEBACK=crash DOCKER_HTTP_HOST_COMPAT=1 PATH=/usr/libexec/docker:/usr/bin:/usr/sbin HTTPS_PROXY=https://127.0.0.1:8118/ NO_PROXY=localhost,172.16.0.0/16,192.168.0.0/16,127.0.0.1,10
.10.0.0/16,10.244.0.0/16
```

#### 2.2安装kubeadm, kubelet和kubectl
使用yum安装:
```bash
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && sudo systemctl start kubelet
```
以上操作需要在，**所有**机器上安装。

###  3. 配置Msater节点

通过kubeadm init命令来初始化，指定kubernetes版本，并设置pod-network-cidr(使用哪种network addon),这里我们使用flannel,`为了flannel正确工作，--pod-network-cidr=10.244.0.0/16必须传递给kubeadm init`
```bash
$ kubeadm init --kubernetes-version=v1.9.0 --pod-network-cidr=10.244.0.0/16
```
> 如果初始化失败,我们使用`kubeadm reset`重置一下
> `kubeadm init`将首先运行一系列预先检查，以确保机器准备运行Kubernetes。它会暴露警告并退出错误。然后它将下载并安装集群数据库和Image组件。

我们会看到如下输出:
```bash
[root@master ~]# kubeadm init --kubernetes-version=v1.9.0 --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.9.0
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
        [WARNING FileExisting-crictl]: crictl not found in system path
        [WARNING HTTPProxy]: Connection to "https://192.168.90.89:6443" uses proxy "http://127.0.0.1:8118". If that is not intended, adjust your proxy settings
        [WARNING HTTPProxyCIDR]: connection to "10.96.0.0/12" uses proxy "http://127.0.0.1:8118". This may lead to malfunctional cluster setup. Make sure that Pod and Services IP ranges specified correctly as exceptions in proxy configuration
[preflight] Starting the kubelet service
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.90.89]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
[init] This might take a minute or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 119.502833 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node master as master by adding a label and a taint
[markmaster] Master master tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: dcc11d.57ec9e4959b413d4
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token dcc11d.57ec9e4959b413d4 192.168.90.89:6443 --discovery-token-ca-cert-hash sha256:7efc53fc16245ba8fe6d133ac1fed91cb2df50dcc270bbf325a628ef9e8dd61a
```

我们看到最后有一串:
```bash
  kubeadm join --token dcc11d.57ec9e4959b413d4 192.168.90.89:6443 --discovery-token-ca-cert-hash 256:7efc53fc16245ba8fe6d133ac1fed91cb2df50dcc270bbf325a628ef9e8dd61a

```
这是node节点加入集群,需要使用到的

#### 3.1 配置kubeconfig
```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
```

#### 3.2 安装网络插件
必须安装pod网络插件，以便pod可以相互通信。
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
```
一旦安装了pod网络，通过检查kube-dns pod 是否正在运行来确认它是否在工作
`kubectl get pods --all-namespaces`一旦kube-dns pod 启动并运行，我们就可以添加node了

```bash
[root@master ~]# kubectl get pods --all-namespaces        
NAMESPACE     NAME                             READY     STATUS    RESTARTS   AGE
kube-system   etcd-master                      1/1       Running   0          8h
kube-system   kube-apiserver-master            1/1       Running   0          8h
kube-system   kube-controller-manager-master   1/1       Running   0          8h
kube-system   kube-dns-6f4fd4bdf-jwzn4         3/3       Running   0          8h
kube-system   kube-flannel-ds-n74md            1/1       Running   0          8h
kube-system   kube-flannel-ds-vdtdd            1/1       Running   2          3h
kube-system   kube-proxy-clw4n                 1/1       Running   0          8h
kube-system   kube-proxy-qpwq4                 1/1       Running   0          3h
kube-system   kube-scheduler-master            1/1       Running   0          8h
```

到这里,master就配置完成了.

### 4. node加入集群
登陆node1,执行下面命令:
```bash
  kubeadm join --token dcc11d.57ec9e4959b413d4 192.168.90.89:6443 --discovery-token-ca-cert-hash 256:7efc53fc16245ba8fe6d133ac1fed91cb2df50dcc270bbf325a628ef9e8dd61a

```

顺利的话可以看到类似如下的提示:
```bash
Run 'kubectl get nodes' on the master to see this node join the cluster.
```

我们回到master执行命令查看:
```bash
[root@master ~]# kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    8h        v1.9.2
node1     Ready     <none>    3h        v1.9.2
```

可以看到节点已经加入了，并且是正常的ready状态。 
至此，整个集群的配置完成，可以开始使用了。

----------
###  资料
感谢文章作者给予参考借鉴
[使用kubeadm部署k8s 1.9](http://blog.csdn.net/u012375924/article/details/78987263)

###  遇到的问题
1. 时间不同步,导致kubeadm  join失败
2. 提示报错`The connection to the server localhost:8080 was refused - did you specify the right host or port?`
3. 在node节点执行`kubectl get pods -n kube-system`报错,和上面一致
4. token过期
   创建永不过期的token
   `kubeadm token create --ttl 0`
   `kubeadm list`
   [k8s的master和node节点](https://blog.frognew.com/2017/01/kubernetes-master-and-node.html)