# _Installing Kubernetes (K8s) on Ubuntu server 22.04 LTS_

We are going to use 

- Containerd as Container runtime interface(CRI)
- Calico as Container Network Interface (CNI)
- Update Ubuntu OS for all node and master
```sh
apt update && apt upgrade && uname -a
```
**Step 1: Setup static IP in every Master and Worker Node**
- Make sure you have only one IP othere wise you have to use --apiserver-advertise-address in kubeadm init command
```sh
vim /etc/netplan/00-installer-config.yaml
```
```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      addresses:
      - 192.168.50.131/24
      nameservers:
        addresses:
        - 8.8.8.8
        - 8.8.4.4
        search: []
      routes:
      - to: default
        via: 192.168.50.1
  version: 2
```
```sh
netplan apply
hostname -I
ip a s
```

**Step 2: Disable Firewall**

```sh
ufw disable
ufw status
```

**Step 3: Host Name Setup**

```sh
hostnamectl set-hostname k8s-master
hostnamectl set-hostname k8s-worker1
hostnamectl set-hostname k8s-worker2
printf "\n#Kubernets hostname configuration\n192.168.50.131 k8s-master\n192.168.50.132 k8s-worker1\n192.168.50.133 k8s-worker2\n\n" >> /etc/hosts
cat /etc/hosts
```

**Step 4: Root SSH Login**
```sh
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
systemctl restart ssh
sudo su -
passwd 
```

**Step 5: Swap partition OFF**
```sh
swapoff -a
cat /etc/fstab  <<<< '#' the swap partition on fstab file
mount -a 
free -h 
```
**Step 6: Kernel modules for Container runtime interface(CRI)**
```sh
printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/k8s-cri.conf
modprobe overlay
modprobe br_netfilter
lsmod | grep overlay && lsmod | grep br_netfilter
```

**Step 7: Kernel parameter for Container runtime interface(CRI)**
```sh
printf "net.bridge.bridge-nf-call-iptables = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\nnet.ipv4.ip_forward = 1\n" >> /etc/sysctl.d/99-kubernets-cri.conf 
sysctl --system
```

**Step 8: Installing containerd From the official binaries**

**_Installing Containerd_**
```sh
wget https://github.com/containerd/containerd/releases/download/v1.6.23/containerd-1.6.23-linux-amd64.tar.gz -P /tmp/
tar Cxzvf /usr/local /tmp/containerd-1.6.23-linux-amd64.tar.gz
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -P /etc/systemd/system/
systemctl daemon-reload && systemctl enable --now containerd
```
**_Installing runc_**
```sh
wget https://github.com/opencontainers/runc/releases/download/v1.1.9/runc.amd64 -P /tmp/
install -m 755 /tmp/runc.amd64 /usr/local/sbin/runc
runc --version
```
**_Installing CNI plugins_**
```sh
wget https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-amd64-v1.3.0.tgz -P /tmp/
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin /tmp/cni-plugins-linux-amd64-v1.3.0.tgz
```
**_Set the cgroup driver for runc to systemd_**
```sh
mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml   <<<<<<<<<<< manually edit and change systemdCgroup to true
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
cat /etc/containerd/config.toml | grep SystemdCgroup
systemctl restart containerd
```
**<u>_Warning Handeling_</u>**
- when we initialize cluster with kubeadm we might get warning like below
```
     [W0909 11:36:07.308518    3646 checks.go:835] detected that the sandbox image "registry.k8s.io/pause:3.6" of the container runtime is inconsistent with that used by kubeadm. It is recommended that using "registry.k8s.io/pause:3.9" as the CRI sandbox image.
```
- Replace sandbox_image = "registry.k8s.io/pause:3.6" to sandbox_image = "registry.k8s.io/pause:3.9" in /etc/containerd/config.toml file
```
sandbox_image = "registry.k8s.io/pause:3.6" >>>> sandbox_image = "registry.k8s.io/pause:3.9"
```

**_Interacting with containerd via CLI_**
```sh
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.28.0/crictl-v1.28.0-linux-amd64.tar.gz -P /tmp/
tar Cxzvf /usr/bin/ /tmp/crictl-v1.28.0-linux-amd64.tar.gz
which crictl
crictl --version
containerd --version
vim /etc/crictl.yaml

runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: false
pull-image-on-create: false

crictl ps
crictl ps -a
crictl pods
crictl images
crictl rmp
crictl rmi
```

**Step 9: Installing Kubernets**

- At first restart all nodes including master and check all the installation and configured kernel module and parameter working properly. You can use below command to use check it.
```sh
lsmod | grep overlay && lsmod | grep br_netfilter && systemctl status containerd && runc --version && sysctl --system && ll /opt/cni/bin && cat /etc/containerd/config.toml | grep SystemdCgroup && cat  /etc/containerd/config.toml | grep sandbox
```

**_For v1.26.8_**
```sh
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/k8s.gpg
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt update
apt install curl apt-transport-https -y
apt-get install -y kubelet=1.26.8-00 kubeadm=1.26.8-00 kubectl=1.26.8-00
apt-mark hold kubelet kubeadm kubectl
```
**_For v1.27.5_**
```sh
apt install curl apt-transport-https -y
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt update
apt install curl apt-transport-https -y
apt install -y kubeadm=1.27.5-00 kubelet=1.27.5-00 kubectl=1.27.5-00
apt-mark hold kubelet kubeadm kubectl
```
**k8s New repository**
[Details](https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/)
```
There are three significant differences that you should be aware of:

    There's a dedicated package repository for each Kubernetes minor release. For example, repository called core:/stable:/v1.28 only hosts packages for stable Kubernetes v1.28 releases. This means you can install v1.28.0 from this repository, but you can't install v1.27.0 or any other minor release other than v1.28. Upon upgrading to another minor version, you have to add a new repository and optionally remove the old one
    There's a difference in what cri-tools and kubernetes-cni package versions are available in each Kubernetes repository
      - These two packages are dependencies for kubelet and kubeadm
      - Kubernetes repositories for v1.24 to v1.27 have same versions of these packages as the Google-hosted repository
      - Kubernetes repositories for v1.28 and onwards are going to have published only versions that are used by that Kubernetes minor release
            -Speaking of v1.28, only kubernetes-cni 1.2.0 and cri-tols v1.28 are going to be available in the repository for Kubernetes v1.28
            -Similar for v1.29, we only plan on publishing cri-tools v1.29 and whatever kubernetes-cni version is going to be used by Kubernetes v1.29
    The revision part of the package version (the -00 part in 1.28.0-00) is now autogenerated by the OpenBuildService platform and has a different format. The revision is now in the format of -x.y, e.g. 1.28.0-1.1
```
**_For v1.27.5_from_new_repository_**
```sh
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.27/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.27/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

apt update
apt install curl apt-transport-https -y
apt list kubeadm
apt list kubelet
apt list kubectl
apt show kubectl -a

apt install aptitude -y
aptitude versions kubectl

sudo apt install kubeadm=1.27.5-1.1 kubelet=1.27.5-1.1 kubectl=1.27.5-1.1
apt-mark hold kubelet kubeadm kubectl
```

**Step 10: Initialize cluster using containerd as container runtime (Only in Master node)**
- At first pull all required images 
```sh
kubeadm config images list
kubeadm config images list --image-repository registry.k8s.io/kubernetes
kubeadm config images pull
kubeadm config print init-defaults
```
- Now initiat the Cluster
```sh
kubeadm init --pod-network-cidr 10.10.18.0/16 --kubernetes-version 1.26.8 --node-name k8s-master
kubeadm init --pod-network-cidr 10.244.0.0/16 --kubernetes-version 1.27.5 --node-name k8s-master
```
- If you have More then one IP Address in your master node use below command
```sh
kubeadm init --apiserver-advertise-address <your_master_node_desired_IP> --pod-network-cidr 10.10.18.0/16 --kubernetes-version 1.26.8 --node-name k8s-master
```

**_Create kubeconfig file to use kubectl command_**
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
- Store kubeadm join from output of kubeadm init command
```sh
sudo kubeadm join 172.16.4.90:6443 --token 7i34jm.q8enu8wxvfic9s8k \
        --discovery-token-ca-cert-hash sha256:202117e62f133323eff707919ec512eef466a59a29454c4ee320a0626ff42c05
```
**Step 11: Installing Calico_v3.25 as Container Network Interface (CNI) plugins for cluster networking(Only in Master node)**
- [Quickstart_Guide](https://docs.tigera.io/calico/3.25/getting-started/kubernetes/quickstart)
```sh
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml
kubectl get all -n tigera-operator

wget https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml
vim custom-resources.yaml <<<<<< match CIDR wirh --pod-network-cidr for pods if its custom
kubectl apply -f custom-resources.yaml
watch kubectl get pods -n calico-system
```
**<u>Install calicoctl as a kubectl plugin on a single host</u>**
```sh
curl -L https://github.com/projectcalico/calico/releases/download/v3.25.0/calicoctl-linux-amd64 -o kubectl-calico
mv kubectl-calico /usr/local/sbin/
chmod 755 /usr/local/sbin/kubectl-calico
kubectl calico -h
kubectl calico node status
```
**_Node's Cluster IP from Calico_**
```sh
hostname -I
ip a s | grep calico
```
- Now you can join your worker node to the mater by running kubeadm join command
- Verify node status and cluster info
```sh
kubectl cluster-info
kubectl get nodes
```
**Step 12: Installing Kubernetes (K8s) Metrics Server (Only in Master node)**
```sh
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
vim components.yaml
```
- Now add this two line in components.yaml file in the right place as shown below
```yaml
hostNetwork: true #set this line
- --kubelet-insecure-tls  #set this line
```
```yaml
    spec:
      hostNetwork: true #set this line
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls  #set this line
        image: registry.k8s.io/metrics-server/metrics-server:v0.6.4
```
```sh
kubectl apply -f components.yaml
kubectl get pods -n kube-system
kubectl top nodes
kubectl top pod -n kube-system
```

**Step 13: Configure kubectl bash completion with alias (Only in Master node)**
```bash
source <(kubectl completion bash) # set up autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
echo 'alias ks=kubectl' >>~/.bashrc
echo 'alias crt=crictl' >>~/.bashrc
echo 'alias clr=clear' >>~/.bashrc
echo 'complete -F __start_kubectl ks' >>~/.bashrc
kubectl completion bash >/etc/bash_completion.d/kubectl
```
**Step 14: Configure vim for YAML**
```bash
vim ~/.vimrc
```
```
set nocompatible
filetype plugin indent on
set title
set number ruler
set noswapfile
set noshowmode
set expandtab
set noshowcmd
set autoindent smartindent
set sts=2 st=2 sw=2 et ai si
set backspace=indent,eol,start
set bg=dark
syntax enable
set cursorline
```
**Step 15: Create a Storage Classes for local storage**
```bash
vim local-storage-sc.yaml
```
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: Immediate
```
```bash
kubectl apply -f local-storage-sc.yaml
kubectl get sc
```
**Step 16: Configure NFS Server**
- Server_Package: nfs-kernel-server
- service: nfs-server
- port: 2049/tcp
- config_file: /etc/exports
- shared_dir: /storageVol
- Permission: NFSUser>NFSNobody UID 65534 Ownership and Group Ownership
```bash
apt install -y nfs-kernel-server
mkdir /k8snfs
printf "/k8snfs		*(rw,sync,no_subtree_check,no_root_squash,no_all_squash,insecure)\n" >> /etc/exports
exportfs -avr
systemctl enable --now nfs-server
systemctl restart nfs-server
systemctl enable nfs-server
showmount -e localhost
grep nfs /etc/passwd
chown 65534:65534 /k8snfs
```
- Install client package in every worker node in cluster
```bash
apt install -y nfs-common
mount -t nfs nfs-server-IP:/k8snfs /mnt
umount /mnt
```