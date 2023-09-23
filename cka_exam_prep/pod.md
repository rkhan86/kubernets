
# How to create or run a pod

- Create a NameSpace
```sh
ks create ns rzk
```
- Run a pod
```sh
kubectl run webapp --image=nginx -n rzk
```
- To see the pods status 
```sh
ks get pods -n rzk
ks get pods -n rzk -o wide
```
- To find out logs 
```sh
ks logs -n rzk pod/webapp
ks logs -n rzk -f pod/webapp
```
- To see the describtion of a pod
```sh
ks describe -n rzk pod webapp
```
- To edit a pod
```sh
ks edit -n rzk pods webapp
```
- to see the running pod in yaml formate 
```sh
ks get -n rzk pods webapp -o yaml
```
- To deleate a pod 
```sh
ks delete -n rzk pod webapp
```
## An sample yaml for pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "MYAPP"
  namespace: default
  labels:
    app: "MYAPP"
spec:
  containers:
  - name: MYAPP
    image: "debian-slim:latest"
    imagePullPolicy: Always
    resources:
      limits:
        cpu: 200m
        memory: 500Mi
      requests:
        cpu: 100m
        memory: 200Mi
    command:
      - "nginx -c deamon off"
    args:
      - "nginx -c deamon off"
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: MYAPP
          key: DB_HOST 
    ports:
    - name:  http
      containerPort:  80 
      protocol: tcp
    volumeMounts:
    - name: localtime
      mountPath: /etc/localtime
  volumes:
    - name: localtime
      hostPath:
        path: /usr/share/zoneinfo/Asia/Shanghai
  restartPolicy: Always
```