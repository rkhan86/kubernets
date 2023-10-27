# POD Labele and Seletor

## POD Labele

- Labels are key/value pairs that are attached to objects such as Pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system.
- labels do not provide uniqueness. In general, we expect many objects to carry the same label(s).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
```

## Label selectors

- Via a label selector, the client/user can identify a set of objects. The label selector is the core grouping primitive in Kubernetes.

- Example from Deployment's lable

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels: # Labels for Deployment
    app: dep1
    environment: production
  name: dep1
  namespace: cka
spec:
  replicas: 5
  selector: # Selector to match POD's labels
    matchLabels:
      run: webhost
  template:
    metadata:
      labels: # Labels for POD
        run: webhost
    spec:
      containers:
        - image: nginx
          name: nginx
          ports:
            - containerPort: 80
```

- Example from Service's lable

```yaml
apiVersion: v1
kind: Service
metadata:
  labels: # Labels for Service
    app: nodeport
  name: nodeport
  namespace: cka
spec:
  ports:
    - name: http
      nodePort: 31333
      port: 80
      protocol: TCP
      targetPort: 80
  selector: # Selector to match POD's labels
    run: webhost
  type: NodePort
status:
  loadBalancer: {}
```

### [Well-Known Labels, Annotations and Taints](https://kubernetes.io/docs/reference/labels-annotations-taints/)
