```sh
kubectl run webhost -n rzk --image=nginx --port=80 --labels='app=webhost' --dry-run=client -o yaml
```

```sh
ks describe nodes k8s-master | grep Taints
Taints:             node-role.kubernetes.io/control-plane:NoSchedule

```

```sh
kubectl get nodes --show-labels
```

```
k8s-master    Ready    control-plane   52d   v1.27.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
k8s-worker1   Ready    <none>          52d   v1.27.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-worker1,kubernetes.io/os=linux
k8s-worker2   Ready    <none>          52d   v1.27.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-worker2,kubernetes.io/os=linux
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    app: webhost
  name: webhost
  namespace: rzk
spec:
  containers:
    - image: nginx
      imagePullPolicy: IfNotPresent/Always/Never
      name: webhost
      ports:
        - name: http
          containerPort: 80
      resources:
        requests:
          memory: "128Mi"
          cpu: "100m"
        limits:
          memory: "256Mi"
          cpu: "200m"
  tolerations:
    - effect: "NoSchedule/NoExecute/PreferNoSchedule"
      operator: "Exists/Equal"
  dnsPolicy: ClusterFirst
  restartPolicy: Always/OnFailure/Never
  nodeSelector:
    node-role.kubernetes.io/control-plane: ""
```

```sh
kubectl label nodes k8s-worker2 disktype=ssd
kubectl get nodes --show-labels
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```
