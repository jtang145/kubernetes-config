# Setup Kubernetes in CentOS 8

## Install Kubernetes steps for master and nodes
### Upgrade Linux Kernel
Follow below steps to upgrade kernel:
```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
yum install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-ml-devel kernel-ml -y
grub2-set-default 0`
```

### (Optional) Set hostname for better operations
```
hostnamectl set-hostname k8s-1
hostnamectl set-hostname k8s-2
```

### Prepare Linux Environment
#### disable firewall & selinux
```
systemctl stop firewalld
systemctl disable firewalld
   
disable selinux      
sed -ie 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
```

#### setup route table
edit /etc/sysctl.conf
```
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf    #路由转发开启
echo "net.bridge.bridge-nf-call-ip6tables = 1" >>/etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >>/etc/sysctl.conf
echo "net.bridge.bridge-nf-call-arptables = 1" >>/etc/sysctl.conf
```      
#### disable all swaps
run

```swapoff -a```

check result:

```sysctl -p```

### Install Docker and Kubernetes components
#### Add K8S repository
run below command to add kubernetes repository: 
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
 [kubernetes]
 name=Kubernetes
 baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
 enabled=1
 gpgcheck=1
 repo_gpgcheck=1
 gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
 exclude=kubelet kubeadm kubectl
 EOF
```

#### Install Docker
Run below commands to install docker
``` 
yum -y update

yum install -y yum-utils

yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo

sudo dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
sudo dnf install docker-ce 
systemctl enable --now docker

systemctl daemon-reload
systemctl restart docker
systemctl enable docker
```

#### Install Kubernetes components
``` 
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
systemctl enable kubelet && systemctl restart kubelet

cat /proc/sys/net/bridge/bridge-nf-call-iptables
cat /proc/sys/net/bridge/bridge-nf-call-ip6tables
echo "KUBELET_EXTRA_ARGS="--fail-swap-on=false"" > /etc/sysconfig/kubelet
systemctl restart kubelet
```       

## Init Kubernetes Master
When master node is ready, run below commands to init master:
``` 
kubeadm init  --pod-network-cidr=10.244.0.0/17 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap  --kubernetes-version=v1.18.6
``` 
after the "kubeadm init" command, copy the result for further steps.
``` 
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml

```                          

For Kubernetes version 1.14 or later, update cgroup driver to systemd:

Edit /etc/docker/daemon.json:
``` 
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}

```
Restart Docker to take effect:
``` 
Restart Docker
systemctl daemon-reload
systemctl restart docker
swapoff -a
sysctl -p
```      

## Join worker nodes
Run the "kubeadm join" commands on each worker node, the token info is from the output from the master node.
e.g.
``` 
kubeadm join 172.16.2.5:6443 --token y1yvp0.d5exuwn9i2tbbq00 \
    --discovery-token-ca-cert-hash sha256:f9bf5fe6e19505b728f844e6838005ece9faad02cb099b5d26ea7d7acf5af323
```                                                                                                       

Check cluster node status by below command on **master node**:
``` 
kubectl get nodes -o wide --all-namespaces

```

## Trouble Shooting
* Work node keeps in "Not ready" state, check log in related work node:
``` 
journalctl -u kubelet
```                  
Go to the last line to check what's reporting there.
* "network plugin is not ready: cni config uninitialized" error on worker node
Edit "/var/lib/kubelet/kubeadm-flags.env" to remove "--network-plugin=cni"