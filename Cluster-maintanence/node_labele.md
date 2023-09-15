
**Add a lable to a Node**

```sh
kubectl label nodes <your-node-name> <key=value>
```

**Deploy a pod to a specific Node**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    ## This label is applied to the Deployment
    type: dev
  name: nginx-deploy
spec:
  replicas: 1
  selector:
     matchLabels:
        ## This label is used to match the Pod to create replicas
        type: dev
  template:
    metadata:
      labels:
        ## This label is applied to the Pod
        type: dev
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
      nodeSelector:
        ## This label is used to deploy the pod on matching nodes
        color: blue
```

**Remove a lable from Node**

```sh
kubectl label node <nodename> <labelname>-
```
**example:**
```sh
kubectl label nodes worker-2.example.com color=blue
kubectl label node worker-2.example.com color-
```