# 一、环境准备
## 1、准备三台虚拟机
操作系统CentOS7.9
资源配置：大于2核2G内存20G硬盘
虚拟机之间内部互通，能访问外网
Docker：19.03.15-ce
k8s：1.21.5
规划：192.168.0.193(master)、192.168.0.19、192.168.0.56
账号密码：root/Hello@163

# 二、虚拟机初始化配置
## 1、备份原始yum文件，配置网络系统yum源
```perl
mv /etc/yum.repos.d/ /etc/yum.repos.d.bak
mkdir /etc/yum.repos.d/
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```
## 2、关闭防火墙、selinux、swap
```perl
systemctl stop firewalld.service
systemctl disable firewalld.service
```
### 将 SELinux 设置为 disabled 模式（相当于将其禁用）
```perl
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
swapoff -a
```
## 3、设置主机名
```perl
hostnamectl set-hostname docker01
hostnamectl set-hostname docker02
hostnamectl set-hostname docker03
```
### 添加地址解析
```perl
echo "192.168.0.193 docker01" >> /etc/hosts
echo "192.168.0.19 docker02" >> /etc/hosts
echo "192.168.0.56 docker03" >> /etc/hosts
```
## 4、设置时钟同步
```perl
systemctl start ntpdate && systemctl enable ntpdate
ntpdate ntp1.aliyun.com
```
## 5、内核参数调优
调整内核参数：vi /etc/sysctl.conf
> 将桥接的IPv4流量传递到iptables的链

```perl
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.ipv4.tcp_tw_reuse=1" >> /etc/sysctl.conf
echo "net.ipv4.tcp_tw_recycle=0" >> /etc/sysctl.conf
echo "net.ipv4.ip_local_port_range=40000 60000" >> /etc/sysctl.conf
echo "vm.swappiness=0" >> /etc/sysctl.conf
echo "kernel.pid_max=655350" >> /etc/sysctl.conf
```
## 6、加载如下两个模块
```perl
modprobe ip_vs_rr
modprobe br_netfilter
sysctl -p
```
# 三、安装docker
## 1、添加Docker的yum源
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
## 2、安装container-selinux包（>=2.74版本）
yum -y install container-selinux
## 3、安装docker
yum install -y docker-ce-19.03.15
## 4、设置cgroup驱动
```perl
mkdir -p /etc/docker/
cat > /etc/docker/daemon.json << EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["http://docker.rainbond.cc/"],
  "insecure-registries": ["0.0.0.0/0"],
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "log-opts": {
    "max-file": "10",
    "max-size": "100m"
  }
}
EOF
```
## 5、启动docker并设置开机自启
```perl
systemctl enable docker && systemctl start docker
systemctl enable containerd && systemctl start containerd
```
# 四、安装k8s
## 1、添加K8S的yum源
```perl
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
## 2、安装kubeadm、kubelet、kubectl
```perl
yum -y install kubeadm-1.21.5 kubelet-1.21.5 kubectl-1.21.5
```
## 3、启动设置开机自启
```perl
systemctl enable kubelet && systemctl start kubelet
```

# 五、创建k8s集群
## 1、初始化控制平面节点（仅在master执行）
```perl
kubeadm init \
  --apiserver-advertise-address=192.168.0.193 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version=1.21.5 \
  --service-cidr=10.96.0.0/16 \
  --pod-network-cidr=10.244.0.0/16
```
## 2、拷贝配置文件（仅在master执行）
```perl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
## 3、加入节点
kubeadm join 192.168.180.2:6443 --token  xxxx （在node执行）
## 4、安装网络插件（仅在master执行）
安装flannel
wget github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
安装kubectl apply -f kube-flannel.yaml

或者安装calico
上传文件  
vim calico.yaml
 
安装kubectl apply -f calico.yaml

## 5、查看集群状态（master节点执行）
```perl
kubectl get nodes
```
## 6、查看pod状态
`kubectl get pods --all-namespaces -o wide`
正在初始化
 

# 常用命令
## 获取节点
```perl
kubectl get nodes -o wide
```
## 实时查询nodes状态
```perl
watch kubectl get nodes -o wide
```
## 获取pod
```perl
kubectl get pods --all-namespaces -o wide
```
## 查看镜像列表
```perl
kubeadm config images list
```
## 节点加入集群
```perl
kubeadm token create --print-join-command
```
## 描述node
```perl
kubectl describe node k8s-master
```
## 描述pod
```perl
kubectl describe pod kube-flannel-ds-hs8bq --namespace=kube-system
```

# 六、部署Web UI
```perl
1、上传dashboard.yaml
2、安装kubectl apply -f dashboard.yaml
3、浏览器访问：https://NodeIP:30001
4、创建service account并绑定默认cluster-admin管理员集群角色
kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard-admin
kubectl describe secrets -n kubernetes-dashboard $(kubectl -n kubernetes-dashboard get secret | awk '/dashboard-admin/{print $1}')
5、使用输出的token登录Dashboard
 
```
 


