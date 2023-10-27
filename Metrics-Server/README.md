# _Installing Kubernetes (K8s) Metrics Server (Only in Master node)_

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
        - --kubelet-insecure-tls #set this line
      image: registry.k8s.io/metrics-server/metrics-server:v0.6.4
```

```sh
kubectl apply -f components.yaml
kubectl get pods -n kube-system
kubectl top nodes
kubectl top pod -n kube-system
```
