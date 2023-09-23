# Creating a Deployment
- Create a NameSpace
```sh
ks create ns rzk
```
- Create a deployment 
```sh
ks create deployment dep1 -n rzk --replicas 3  --image nginx 
```

```sh
vim depdemo.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```
```sh
ks apply -f depdemo.yaml
ks get deployments.apps -n rzk 
ks get replicasets.apps -n rzk 
ks get pods -n rzk
ks scale -n rzk --replicas=5 deployment/demo-deployment 
ks get pods -n rzk
ks scale -n rzk --replicas=3 deployment/demo-deployment 
ks get pods -n rzk
```
- To create a deployment yaml 
```sh
ks create deployment dep1 -n rzk --replicas 3  --image nginx --dry-run=client -o yaml
```
- To deleate a Deployment 
```sh
ks delete -n rzk deployments.apps demo-deployment dep1 
```

## An sample yaml for Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  MYAPP
  namespace: default
  labels:
    app:  MYAPP
spec:
  selector:
    matchLabels:
      app: MYAPP
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app:  MYAPP
    spec:
      # initContainers:
        # Init containers are exactly like regular containers, except:
          # - Init containers always run to completion.
          # - Each init container must complete successfully before the next one starts.
      containers:
      - name:  MYAPP
        image:  MYAPP:latest
        imagePullPolicy: Always
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 100m
            memory: 100Mi
        livenessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 5
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /_status/healthz
            port: 80
          initialDelaySeconds: 5
          timeoutSeconds: 2
          successThreshold: 1
          failureThreshold: 3
          periodSeconds: 10
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: MYAPP
              key: DB_HOST
        ports:
        - containerPort:  80
          name:  MYAPP
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

## Updating a Deployment
- **Note:** A Deployment's rollout is triggered if and only if the Deployment's Pod template (that is, .spec.template) is changed, for example if the labels or container images of the template are updated. Other updates, such as scaling the Deployment, do not trigger a rollout.
- Deployment also ensures that only a certain number of Pods are created above the desired number of Pods. By default, it ensures that at most 125% of the desired number of Pods are up (25% max surge).
```yaml
selector:
    matchLabels:
      app: MYAPP
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```
- Exampl as follows
```sh
ks create deployment dep1 -n rzk --replicas 3  --image nginx:1.14.2
ks get deployments.apps -n rzk
ks get pod -n rzk
kubectl set image deployment/dep1 -n rzk nginx=nginx:1.21
kubectl annotate deployments dep1  kubernetes.io/change-cause="set image to nginx v1.21" -n rzk
kubectl rollout status deployment/dep1 -n rzk 
kubectl get rs
kubectl get rs -n rzk
kubectl get replicasets.apps -n rzk
```
## Checking Rollout History of a Deployment
- check the revisions of this Deployment:
```sh
kubectl rollout history deployment/dep1 -n rzk
```
```
deployment.apps/dep1 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
4         kubectl set image deployment/dep1 nginx=nginx:1.22 --namespace=rzk --record=true
5         set image to nginx v1.21
6         back to revison 3 nginx v1.25
```
- To see the details of each revision, run:
```sh
kubectl rollout history deployment/dep1 --revision=3 -n rzk
```
```
deployment.apps/dep1 with revision #3
Pod Template:
  Labels:	app=dep1
	pod-template-hash=dddd59c97
  Containers:
   nginx:
    Image:	nginx:1.25
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```
## Rolling Back to a Previous Revision
- Now you've decided to undo the current rollout and rollback to the previous revision:
```sh
kubectl rollout undo deployment/nginx-deployment -n rzk
```

- Alternatively, you can rollback to a specific revision by specifying it with --to-revision:
```sh
kubectl rollout undo deployment/nginx-deployment --to-revision=3 -n rzk
```