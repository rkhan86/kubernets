# [Node Selector](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/)

- Choose one of your nodes, and add a label to it:

```sh
kubectl label nodes <your-node-name> disktype=ssd
kubectl get nodes --show-labels
```

### Create a pod that gets scheduled to your chosen node

- This pod configuration file describes a pod that has a node selector, disktype: ssd. This means that the pod will get scheduled on a node that has a disktype=ssd label.

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
