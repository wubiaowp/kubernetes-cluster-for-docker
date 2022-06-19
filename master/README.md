**一基础配置**
 
    # 关闭防火墙和selinux 
    systemctl stop firewalld
    systemctl disable firewalld
    setenforce 0
    sed -i.bak 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config


    ## 关闭swap分区
    swapoff -a
    sed -i.bak '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab


    ## 添加下载源
    curl  http://mirrors.aliyun.com/repo/epel-7.repo  > /etc/yum.repos.d/epel.repo
    curl -o /etc/yum.repos.d/docker-ce.repo https://files-cdn.cnblogs.com/files/lemanlai/docker-ce.repo.sh
    yum clean all
    yum makecache fast  -y


    ## 安装Docker
    yum install -y yum-utils device-mapper-persistent-data lvm2
    yum -y install docker-ce
    systemctl enable docker


    ## 替换镜像地址
    mkdir -p /etc/docker/
    cat << EOF > /etc/docker/daemon.json
    {"registry-mirrors": ["http://fl791z1h.mirror.aliyuncs.com"],"exec-opts": ["native.cgroupdriver=systemd"]}
    EOF


    ## 重启配置生效
    systemctl daemon-reload
    systemctl restart docker


    ## 下载镜像
    将本地项目master目录的镜像脚本上传到maste节点：scp pull_master_image.sh root@masterIP:/opt/images/
    cd /opt/images/
    sh pull_master_image.sh
    docker images[这里coredns镜像要单独打个tag]
    docker tag k8s.gcr.io/coredns:v1.8.6 k8s.gcr.io/coredns/coredns:v1.8.6
    docker rmi k8s.gcr.io/coredns:v1.8.6


    ## 将桥接的IPv4流量传递到iptables的链
    cat > /etc/sysctl.d/k8s.conf << EOF
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    net.ipv4.ip_forward = 1
    EOF


    ## 安装kubeadm工具
    cat << EOF >/etc/yum.repos.d/kubernetes.repo
    [Kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=0 
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
           https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg 
    EOF

    yum clean all
    yum makecache fast  -y


    ## 下载kubeadm、kubectl、kubelet
    yum install -y kubeadm-1.23.1 kubelet-1.23.1 kubectl-1.23.1
    systemctl enable kubelet


    ## 设置主机名
    hostnamectl set-hostname kubernetes-master

    ## 设置网络插件
    echo "1" >/proc/sys/net/bridge/bridge-nf-call-iptables

    ## master节点初始化
    kubeadm init --kubernetes-version=v1.23.1 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
    直至底部出现【保存下来后面在node节点执行】 kubeadm join xxxxxxxxxx --token xxxxxxxxxx --discovery-token-ca-cert-hash sha256:xxxxxxxxxx表示成功[保存该文件]


    ## 创建集群配置文件
    mkdir -p $HOME/.kube
    cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    chown $(id -u):$(id -g) $HOME/.kube/config


    ## 查看节点状态[NotReady]
    kubectl get nodes
    

    ## 部署flannel
    vim /etc/hosts 添加 185.199.110.133  raw.githubusercontent.com
    wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    kubectl apply -f kube-flannel.yml
    
    
    ##修改标签名称[可选操作-集群部署完成之后操作]
    kubectl label node kubernetes-node1 node-role.kubernetes.io/node1=node1
    kubectl label node kubernetes-node2 node-role.kubernetes.io/node2=node2
    
