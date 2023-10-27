# _Installing containerd From the official binaries_

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

**_Interacting with containerd via CLI_**

```sh
wget https://github.com/containerd/nerdctl/releases/download/v1.5.0/nerdctl-1.5.0-linux-amd64.tar.gz
tar Cxzvf /usr/local/bin nerdctl-1.5.0-linux-amd64.tar.gz
which nerdctl
nerdctl run -d --name nginx -p 80:80 nginx:alpine
nerdctl ps
nerdctl images
nerdctl logs nginx
```

```sh
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.28.0/crictl-v1.28.0-linux-amd64.tar.gz -P /tmp/
tar Cxzvf /usr/bin/ /tmp/crictl-v1.28.0-linux-amd64.tar.gz
which crictl
crictl --version
containerd --version
vim cat /etc/crictl.yaml

runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: false
pull-image-on-create: false

crictl ps
crictl pods
```
