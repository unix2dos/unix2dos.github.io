---
title: "跟着官网学k8s"
date: 2020-04-06 00:00:00
tags:
- k8s
---

<!-- more -->

2. 1. 安装前要求

服务器要2GB RAM 和 2个 CPU


2. 安装docker

apt update
apt install docker.io



3. 安装Kubernetes

3.1 安装

# ubuntu

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -


cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt update
apt install -y kubelet kubeadm kubectl



# centos


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






3.2 修改网络配置

cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system



3.3  禁止 swap

swapoff -a


4.  kubeadm 创建


kubeadm init 

显示下面就是成功了
Your Kubernetes control-plane has initialized successfully!


kubeadm join 10.128.0.2:6443 --token 90d9u3.8p6gi8701z56ny5z \
    --discovery-token-ca-cert-hash sha256:cf2d3cda573ce07f0c5a0b6a34ff855bb503ba7ac90a5c30291a0342d4418fa4




kubectl get node 
error: no configuration has been provided, try setting KUBERNETES_MASTER environment variable


非 root用户

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

root 用户
 export KUBECONFIG=/etc/kubernetes/admin.conf




root@us:~# kubectl get node
NAME   STATUS     ROLES    AGE   VERSION
us     NotReady   master   10m   v1.18.2


此处的NotReady是因为网络还没配置.



5. 配置网络
By default, Calico uses 192.168.0.0/16 as the Pod network CIDR, though this can be configured in the calico.yaml file. 


kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml



 kubectl get pods --all-namespaces
可以看到calico安装上了, 过一会就能看到状态是 Running 了



kubectl get node  也能看到状态是Ready 了




6. 部署图形界面

6.1 下载

  https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/



kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml


部署完毕后, 执行kubectl get pods --all-namespaces 可以看到有 dashboard



6.2  创建用户


vi dashboard-adminuser.yaml



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




kubectl apply -f dashboard-adminuser.yaml



显示 clusterrolebinding.rbac.authorization.k8s.io/admin-user created



6.3 访问dashboard

https://34.66.187.249:6443/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy



  "message": "services \"https:kubernetes-dashboard:\" is forbidden: User \"system:anonymous\" cannot get resource \"services/proxy\" in API group \"\" in the namespace \"kubernetes-dashboard\"",




这个是因为kubernetes基于安全性的考虑，浏览器必须要一个根证书，防止中间人攻击



#生成crt
grep 'client-certificate-data' /etc/kubernetes/admin.conf | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.crt
#生成key文件
grep 'client-key-data' /etc/kubernetes/admin.conf | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.key
# 生成p12证书文件（证书的生成和导入需要一个密码）
openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-client"

第三条命令生成证书时会提示输入密码, 建议输入密码.


kubecfg.p12即需要导入客户端机器的证书. 将证书拷贝到客户端机器上(自己的电脑), 导入即可(登录钥匙链并且始终信任). 



再次登录时会提示选择证书, 确认后会提示输入当前用户名密码(注意是电脑的用户名密码).


然后需要输入token, 用下面指令获取token, 输入即可

kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')  


坑爹
执行
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')




7. 添加Worker节点

kubeadm join 10.33.30.92:6443 --token abcdef.0123456789abcdef \ --discovery-token-ca-cert-hash sha256:2883b1961db36593fb67ab5cd024f451b934fc0e72e2fa3858dda3ad3b225837

如果忘记了, 使用







9. 遇到的问题

+ 如何停止 kube-schedule
systemctl stop kubelet



+ dash无法查看的问题

nodes is forbidden: User "system:serviceaccount:kubernetes-dashboard:kubernetes-dashboard" cannot list resource "nodes" in API group "" at the cluster scope

其实很明显就是用户system:serviceaccount:kubernetes-dashboard:kubernetes-dashboard没有相关权限
修改dashboard-adminuser.yaml 如上文所示即可



+     加入 master节点出错
    [ERROR DirAvailable--etc-kubernetes-manifests]: /etc/kubernetes/manifests is not empty

kubeadm reset  清理



+   加入 master节点出错

error execution phase preflight: couldn't validate the identity of the API Server: abort connecting to API servers after timeout of 5m0s

kubeadm token create --ttl 0

openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'



+  打开 dashboard报错
"no endpoints available for service \"kubernetes-dashboard\""
kubectl get pods --all-namespaces 发现
kubernetes-dashboard ContainerCreating



安装calico网络后解决

+ 不同公网服务器集群


https://github.com/kubernetes/kubeadm/issues/1390


iptables -t nat -A OUTPUT -d 10.128.0.2 -j DNAT --to-destination 34.66.187.249










10. # 参考教程:

+ https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/ 官方教程

+ https://juejin.im/post/5d7fb46d5188253264365dcf   安装教程