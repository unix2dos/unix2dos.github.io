---
title: k8s的安装和使用
tags:
  - k8s
categories:
  - 2-linux系统
  - k8s
abbrlink: caa8b60
date: 2020-02-23 00:00:00
---

# 1. 安装

### 1.1 安装前要求

Master服务器要2GB RAM 和 2个 CPU, docker和 k8s 在 master 和 node 节点都需要安装.

<!-- more -->

### 1.2 安装docker

```bash
apt update
apt install docker.io
```

### 1.3 安装k8s

```bash
# ubuntu 安装
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt update
apt install -y kubelet kubeadm kubectl


# centos 安装
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm kubectl
```

修改网络配置

```bash
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system
```

禁止 swap

```bash
swapoff -a
```



# 2. 操作

### 2.1 kubeadm 创建

```bash
kubeadm init

# 显示下面就是成功了
Your Kubernetes control-plane has initialized successfully!
kubeadm join 10.128.0.2:6443 --token essacx.dirj093suimneobu \
    --discovery-token-ca-cert-hash sha256:ccfba722f83313d31e27249300501ea88ef0560d7674b7a063fca5b2a3db357c
```

如果安装过程出现问题, 重置

```bash
kubeadm reset
systemctl stop kubelet
systemctl stop docker
rm -rf /var/lib/cni/
rm -rf /var/lib/calico/
rm -rf /var/lib/kubelet/*
rm -rf /etc/cni/

systemctl start docker
systemctl start kubelet
kubeadm init
```



### 2.2 获取 node 信息

```bash
kubectl get nodes -o wide
# 报错
error: no configuration has been provided, try setting KUBERNETES_MASTER environment variable

# 非 root用户
mkdir -p HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf HOME/.kube/config
sudo chown (id -u):(id -g) $HOME/.kube/config

#root 用户
export KUBECONFIG=/etc/kubernetes/admin.conf
```

再次获取可看到状态

```bash
kubectl get nodes # 此处的NotReady是因为网络还没配置.
NAME   STATUS     ROLES    AGE   VERSION
us     NotReady   master   10m   v1.18.2
```



### 2.3 配置网络

选一个即可

+ 安装Calico

```bash
wget https://docs.projectcalico.org/v3.11/manifests/calico.yaml


# calico.yaml 文件添加以下二行
- name: IP_AUTODETECTION_METHOD
	value: "interface=ens.*"  # ens 根据实际网卡开头配置


kubectl delete -f calico.yaml
kubectl apply -f calico.yaml
```

+ 安装 flannel

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl delete -f kube-flannel.yml
kubectl apply -f kube-flannel.yml
```



### 2.4 添加Worker节点

```bash
# 获取 nodes
kubectl get nodes -o wide

# 删除 nodes
kubectl delete nodes tencent

# 在被删除的node节点清空集群信息
kubeadm reset

# 在master节点查看集群的token值(在 master 机器操作)
kubeadm token create --print-join-command

# 将node节点重新添加到k8s集群中(在 node 机器操作)
kubeadm join 10.128.0.2:6443 --token oa5h63.wylpy4upqzp5nxl4     --discovery-token-ca-cert-hash sha256:182bf7a949b7cad1511df0b38e340dde0196084522132ab6a32da1ae9e4c9d1a
```



# 3. 应用部署

### 3.1 nginx

```bash
vi nginx-pod.yaml

apiVersion: v1      # 描述文件所遵循KubernetesAPI的版本
kind: Pod           # 描述的类型是pod
metadata:
  name: nginx-pod   # pod的名称
  labels:           # 标签
    app: nginx-pod
    env: test
spec:
  containers:
    - name: nginx-pod     # 容器名
      image: nginx:1.15   # 镜像名称及版本
      imagePullPolicy: IfNotPresent   # 如果本地不存在就去远程仓库拉取
      ports:
        - containerPort: 80   # pod对外端口
  restartPolicy: Always
```
应用和删除
```bash
kubectl apply -f nginx-pod.yaml  # 应用
kubectl delete -f nginx-pod.yaml # 删除
```

注意, nginx 是在 node 节点上面运行的, 而不是 master, 可以通过 docker ps 查看



+ 访问nginx

  想要访问到pod中的服务, 最简单的方式就是通过端口转发, 执行如下命令, 将宿主机的`9999`端口与nginx-pod的`80`端口绑定:

  ```bash
  kubectl port-forward --address 0.0.0.0 nginx-pod 9999:80
  ```

  然后在 master 身上访问

  ```bash
  curl localhost:9999
  ```

  



# 8. 部署图形界面

### 8.1 下载

  https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

kubectl get pods --all-namespaces 
# 部署完毕后, 执行可以看到有 dashboard
```



### 8.2 创建用户

+ vi dashboard-adminuser.yaml

  按照下面的去写, 否则可能有权限问题

```bash
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```



```bash
kubectl apply -f dashboard-adminuser.yaml
# 显示 clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```



### 8.3 访问dashboard

https://34.66.187.249:6443/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy

报错  "message": "services \"https:kubernetes-dashboard:\" is forbidden: User \"system:anonymous\" cannot get resource \"services/proxy\" in API group \"\" in the namespace \"kubernetes-dashboard\"",


这个是因为kubernetes基于安全性的考虑，浏览器必须要一个根证书，防止中间人攻击

```bash
# 生成crt
grep 'client-certificate-data' /etc/kubernetes/admin.conf | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.crt

# 生成key文件
grep 'client-key-data' /etc/kubernetes/admin.conf | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.key

# 生成p12证书文件（证书的生成和导入需要一个密码, 建议输入密码）
openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-client"
```



kubecfg.p12即需要导入客户端机器的证书. 将证书拷贝到客户端机器上(自己的电脑), 导入即可(登录钥匙链并且始终信任). 
再次登录时会提示选择证书, 确认后会提示输入当前用户名密码(注意是电脑的用户名密码)

```bash
# 执行, 复制 token 到浏览器即可
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```



# 9. 错误问题总结

### 9.1 如何停止 kube-schedule

```bash
systemctl stop kubelet
```

### 9.2 dashboard无法查看的问题

nodes is forbidden: User "system:serviceaccount:kubernetes-dashboard:kubernetes-dashboard" cannot list resource "nodes" in API group "" at the cluster scope

其实很明显就是用户system:serviceaccount:kubernetes-dashboard:kubernetes-dashboard没有相关权限

修改dashboard-adminuser.yaml 如上文所示即可

### 9.3 镜像拉取失败 GFW

Failed to create pod sandbox: rpc error: code = Unknown desc = failed pulling image "k8s.gcr.io/pause:3.1": Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

MountVolume.SetUp failed for volume "kube-proxy" : failed to sync configmap cache: timed out waiting for the condition

Failed to pull image "k8s.gcr.io/kube-proxy:v1.18.2": rpc error: code = Unknown desc = Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

```bash
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
docker tag  registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.18.2 
docker tag  registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.18.2   k8s.gcr.io/kube-proxy:v1.18.2
```

### 9.4  calico无法启动

 Readiness probe failed: calico/node is not ready: felix is not ready: Get http://localhost:9099/readiness: dial tcp 127.0.0.1:9099: connect: connection refused

修改calico.yaml, 增加`IP_AUTODETECTION_METHOD`环境变量, 在 IP 的下面

```bash
- name: IP
	value: "autodetect"
- name: IP_AUTODETECTION_METHOD
	value: "interface=eth.*"
```

重启calico

```bash
kubectl delete -f calico.yaml
kubectl apply -f calico.yaml
```



### 9.5 node 无法 join

 [ERROR FileContent–proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```



# 10. 常用命令

```bash
# 查看状态
kubectl get nodes --all-namespaces -o wide
kubectl get pods --all-namespaces -o wide


# 查看 pod 信息
kubectl describe pods name
kubectl describe pods -n kube-system name


# 获取 token, 加入
kubeadm token create --print-join-command


# 清理
kubeadm reset
rm -rf /var/lib/cni/ && rm -rf /var/lib/calico/ && rm -rf /var/lib/kubelet/ && rm -rf /etc/cni/
kubeadm init
```



# 11. 参考教程

+ https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
+ https://juejin.im/post/5d7fb46d5188253264365dcf
+ https://juejin.im/post/5e1e96ef6fb9a02fbd378588
