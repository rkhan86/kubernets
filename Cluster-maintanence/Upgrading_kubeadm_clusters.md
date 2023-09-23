# <u>_Upgrading kubeadm clusters_</u>

 - **Changing the package repository to pkgs.k8s.io for both Control plane and worker node** 
```sh
cd /etc/apt/keyrings/  && \
rm -fr kubernetes-apt-keyring.gpg && \
cd /etc/apt/sources.list.d/ && \
rm -fr kubernetes.list 
apt update
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.27/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.27/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
- **Take backup of ETCD** 

```sh
etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshort save <Directory_location_of_backup_file>
```

## <u>Upgrading Control plane</u> 

**Determine which version to upgrade to**
```sh
apt update
apt-cache madison kubeadm
```

**Upgrading kubeadm and apply upgrade to control plane nodes**
```sh
# Upgrade kubeadm:
apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm='1.27.6-1.1' && apt-mark hold kubeadm
# Verify that the download works and has the expected version:
kubeadm version
# Verify the upgrade plan:
kubeadm config images pull
kubeadm upgrade plan
# Choose a version to upgrade to, and run the appropriate command. For example:
kubeadm upgrade apply v1.27.6
kubectl get nodes
```
**Upgrade kubelet and kubectl**

```sh
# Drain the master node
kubectl drain k8s-master --ignore-daemonsets
# Upgrade the kubelet and kubectl:
apt-mark unhold kubelet kubectl && apt-get update && apt-get install -y kubelet='1.27.6-1.1' kubectl='1.27.6-1.1' && apt-mark hold kubelet kubectl
systemctl restart kubelet
kubectl get nodes
# Uncordon k8s-master
kubectl uncordon k8s-master
```

## <u>Upgrading Linux nodes </u>

**Determine which version to upgrade to**
```sh
apt update
apt-cache madison kubeadm
```
**Upgrading kubeadm and apply upgrade to nodes**
```sh
# Upgrade kubeadm:
apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm='1.27.6-1.1' && apt-mark hold kubeadm
# Verify that the download works and has the expected version:
kubeadm version
# For worker nodes this upgrades the local kubelet configuration:
kubeadm upgrade node
kubectl get nodes
```
**Upgrade kubelet and kubectl**

```sh
# Drain the k8s-worker1 from Control plane
kubectl drain k8s-worker1 --ignore-daemonsets
kubectl drain k8s-worker2 --ignore-daemonsets --delete-emptydir-data
# Upgrade the kubelet and kubectl:
apt-mark unhold kubelet kubectl && apt-get update && apt-get install -y kubelet='1.27.6-1.1' kubectl='1.27.6-1.1' && apt-mark hold kubelet kubectl
systemctl restart kubelet
kubectl get nodes
# Uncordon k8s-worker1 from Control plane
kubectl uncordon k8s-workerX
```

**Verify all is Working**

```sh
kubectl get all -A -o wide
kubectl get all -A --show-labels
ks calico node status
```