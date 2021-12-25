#基于Docker本地运行Kubernetes
**先决条件**
1. 你必须拥有一台安装有Docker的机器。

2. 你的内核必须支持 memory and swap accounting 。确认你的linux内核开启了如下配置： 
   
    CONFIG_RESOURCE_COUNTERS=y
   
    CONFIG_MEMCG=y
   
    CONFIG_MEMCG_SWAP=y
    
    CONFIG_MEMCG_SWAP_ENABLED=y
   
    CONFIG_MEMCG_KMEM=y

notes:cat boot/config-XXXX   

3. 以命令行参数方式，在内核启动时开启 memory and swap accounting 选项：

    GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"

notes: vim /etc/default/grub   **如果已有内容，加在后面**

centOS系统需要grub2-mkconfig -o /boot/grub2/grub.cfg

reboot

注意：以上只适用于GRUB2。通过查看/proc/cmdline可以确认命令行参数是否已经成功

**第一步：运行Etcd**

docker run --net=host -d gcr.io/google_containers/etcd:2.0.12 /usr/local/bin/etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data

**注：**
   docker安装k8s链接不到国外镜像
解决方案：

docker pull mirrorgooglecontainers/etcd:2.0.12

docker tag docker.io/mirrorgooglecontainers/etcd:2.0.12 gcr.io/google_containers/etcd:2.0.12

docker images

第二步：启动master
docker run \
--volume=/:/rootfs:ro \
--volume=/sys:/sys:ro \
--volume=/dev:/dev \
--volume=/var/lib/docker/:/var/lib/docker:ro \
--volume=/var/lib/kubelet/:/var/lib/kubelet:rw \
--volume=/var/run:/var/run:rw \
--net=host \
--pid=host \
--privileged=true \
-d \
gcr.io/google_containers/hyperkube:v1.0.1 \
/hyperkube kubelet --containerized --hostname-override=&quot;127.0.0.1&quot; --address=&quot;0.0.0.0&quot; --api-servers=http://localhost:8080 --config=/etc/kubernetes/manifests


mirrorgooglecontainers/hyperkube:v1.0.1
docker tag docker.io/mirrorgooglecontainers/hyperkube:v1.0.1 gcr.io/google_containers/hyperkube:v1.0.1

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