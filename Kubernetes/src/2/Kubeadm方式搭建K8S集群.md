#Kubeadm方式搭建K8S集群
使用kubeadm方式搭建K8s集群主要分为以下几步

准备三台虚拟机，同时安装操作系统CentOS 7.x

对三个安装之后的操作系统进行初始化操作

在三个节点安装 docker kubelet kubeadm kubectl

在master节点执行kubeadm init命令初始化

在node节点上执行 kubeadm join命令，把node节点添加到当前集群

配置CNI网络插件，用于节点之间的连通【失败了可以多试几次】

通过拉取一个nginx进行测试，能否进行外网测试
#准备环境

在每台虚拟机上执行：

关闭防火墙

systemctl stop firewalld

systemctl disable firewalld

关闭selinux

永久关闭

sed -i 's/enforcing/disabled/' /etc/selinux/config

临时关闭

setenforce 0

关闭swap

临时

swapoff -a

永久关闭

sed -ri 's/.*swap.*/#&/' /etc/fstab

根据规划设置主机名【master节点上操作】

hostnamectl set-hostname k8smaster

根据规划设置主机名【node1节点操作】

hostnamectl set-hostname k8snode1

根据规划设置主机名【node2节点操作】

hostnamectl set-hostname k8snode2

在master添加hosts

cat >> /etc/hosts << EOF

192.168.177.130 k8smaster

192.168.177.131 k8snode1

192.168.177.132 k8snode2

EOF


将桥接的IPv4流量传递到iptables的链

cat > /etc/sysctl.d/k8s.conf << EOF

net.bridge.bridge-nf-call-ip6tables = 1

net.bridge.bridge-nf-call-iptables = 1

EOF

生效

sysctl --system

时间同步

yum install ntpdate -y

ntpdate time.windows.com

#安装Docker/kubeadm/kubelet
所有节点安装Docker/kubeadm/kubelet ，Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker

#安装Docker
首先配置一下Docker的阿里yum源

    cat >/etc/yum.repos.d/docker.repo<<EOF
    [docker-ce-edge]
    name=Docker CE Edge - \$basearch
    baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/\$basearch/edge
    enabled=1
    gpgcheck=1
    gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
    EOF

然后yum方式安装docker

**yum安装**

    yum -y install docker-ce

**查看docker版本**

    docker --version

**启动docker**

    systemctl enable docker

    systemctl start docker

**配置docker的镜像源**

    cat >> /etc/docker/daemon.json << EOF
    {
    "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
    }
    EOF

然后重启docker

    systemctl restart docker

#添加kubernetes软件源

配置yum的k8s软件源

    cat > /etc/yum.repos.d/kubernetes.repo << EOF
    [kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=0
    repo_gpgcheck=0
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF

#安装kubeadm，kubelet和kubectl
指定版本号部署：
**安装kubelet、kubeadm、kubectl，同时指定版本**

    yum install -y kubelet-1.21.3 kubeadm-1.21.3 kubectl-1.21.3
**设置开机启动**

    systemctl enable kubelet

#部署Kubernetes Master【master节点】
在 192.168.13.3 执行，也就是master节点

    kubeadm init --apiserver-advertise-address=192.168.177.130 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.0 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16
**报错：failed to pull image registry.aliyuncs.com/google_containers/coredns:v1.8.0**
阿里云找不到coredns：v1.8.0，此为k8s：v1.21.0以上版本所需
    解决：

    docker pull coredns/coredns:1.8.0
    
    docker tag coredns/coredns:1.8.0 registry.aliyuncs.com/google_containers/coredns:v1.8.0

然后重新部署master节点

生成join：

    kubeadm join 192.168.13.3:6443 --token awyw5o.lt36dq8uonpg1rzd \
    --discovery-token-ca-cert-hash sha256:a9261ff229ba79e7fba43a8425a8d6e4ab88ac283188cc57ceea6a146f0b5cfc

**使用kubectl工具 【master节点操作】**

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

使用kubectl get nodes查看正在运行的节点
![img_1.png](img_1.png)

**加入Kubernetes Node【Slave节点】**
下面我们需要到 node1 和 node2服务器，执行下面的代码向集群添加新节点

执行在kubeadm init输出的kubeadm join命令(注意：直接复制可能会在第二行多出一个.bash)

默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下：

    kubeadm token create --print-join-command

#部署CNI网络插件
上面的状态还是NotReady，下面我们需要网络插件，来进行联网访问

**下载网络插件配置**
    wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
默认镜像地址无法访问，sed命令修改为docker hub镜像仓库。


**添加**

    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
**查看状态【kube-system是k8s中的最小单元】**

    kubectl get pods -n kube-system

如果上述操作完成后，还存在某个节点处于NotReady状态，可以在Master将该**节点删除**
kubectl delete node k8snode1

**然后到k8snode1节点进行重置**

    kubeadm reset
**重置完后在加入**

    kubeadm join 192.168.177.130:6443 --token 8j6ui9.gyr4i156u30y80xf     --discovery-token-ca-cert-hash sha256:eda1380256a62d8733f4bddf926f148e57cf9d1a3a58fb45dd6e80768af5a500

**测试kubernetes集群**
在Kubernetes集群中创建一个pod，验证是否正常运行：

    # 下载nginx 【会联网拉取nginx镜像】
    kubectl create deployment nginx --image=nginx
    # 查看状态
    kubectl get pod
出现Running状态时，表示已经成功运行了

下面我们就需要将端口暴露出去，让其它外界能够访问

**暴露端口**

    kubectl expose deployment nginx --port=80 --type=NodePort
**查看一下对外的端口**

    kubectl get pod,svc

主机浏览器上，访问如下地址

    http://192.168.13.3:30572/