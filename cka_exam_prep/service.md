**<u>Note:</u>** **Selector section need to match with POD or Deployment's lables for both service type to make it work.**

## type: NodePort

- If you set the type field to NodePort, the Kubernetes control plane allocates a port from a range specified by --service-node-port-range flag (default: 30000-32767). Each node proxies that port (the same port number on every Node) into your Service.

```sh
kubectl create service nodeport NAME [--tcp=port:targetPort] [--namespace=NAME] [--nodeport=30000-32767] [--dry-run=server|client|none] [options]
ks create service nodeport ndp1 --tcp 80:80 --namespace rzk --node-port 31333 --dry-run=client -o yaml
ks delete service -n rzk ndp1
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: ndp1
  name: ndp1
  namespace: rzk
spec:
  ports:
    - name: 80-80
      nodePort: 31333
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: ndp1
  type: NodePort
status:
  loadBalancer: {}
```

## type: ClusterIP

- ClusterIP is the default service type in Kubernetes, and it provides internal connectivity between different components of our application. Kubernetes assigns a virtual IP address to a ClusterIP service that can solely be accessed from within the cluster during its creation. This IP address is stable and doesn’t change even if the pods behind the service are rescheduled or replaced.

```sh
kubectl create service clusterip NAME [--tcp=<port>:<targetPort>] [--dry-run=server|client|none] [options]
ks create service clusterip cltrip1 --tcp 80:80 --namespace rzk --dry-run=client -o yaml
ks delete service -n rzk cltrip1
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: ndp1
  name: ndp1
  namespace: rzk
spec:
  ports:
    - name: 80-80
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: ndp1
  type: ClusterIP
status:
  loadBalancer: {}
```
