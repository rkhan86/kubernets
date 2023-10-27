# [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress #entering to POD
    - Egress #leaving POD
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```

### Task:

- Create a new NetworkPolicy named allow-namespace in the existing namespace fubar.</br>
- Ensure that the new Networkpolicy allow PODs in namespace internal to connect port 9000 of pods in namespace fuber
- Further ensure that the new NetworkPolicy:
  - does not allow access to pods, which don't listen on port 9000
  - does not allow access from pods, which are not in namespace internal

```sh
vim policy.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-namespace
  namespace: fuber
spec:
  podSelector: {}
  policyTypes:
    - Ingress #entering to POD
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              project: myproject
      ports:
        - protocol: TCP
          port: 9000
```

```sh
kubectl label ns internal project=myproject
kubectl describe ns internal
kubectl create -f policy.yaml
```
