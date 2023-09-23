- **location: $HOME/.kube/config**

```sh
ks config view 
ks config -h
```

```yaml
apiVersion: v1
kind: Config
preferences: {}
# 
clusters:
- cluster:
# Below is /etc/kubernetes/pki/ca.crt in base64 formate
    certificate-authority-data: QWVGdzB5TXpBNU1EWXdNekl3TkRsYUZ3MHlOREE1TURVd016STFOVEphTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVl
    server: https://192.168.50.131:6443 # Kubernetes control plane is running at
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes # This line join the user to the cluster
users:
- name: kubernetes-admin #admin user name
  user:
# Below is /etc/kubernetes/pki/admin.crt in base64 formate
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJRDJFWGtlejF0eUV3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExV
    
# Below is /etc/kubernetes/pki/admin.key in base64 formate
    client-key-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJRDJFWGtlejF0eUV3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS
    
```

- To convert a file to base64 formate 
```sh
cat ca.crt | base64
```
- To decode a base64 formate to get crt file content 
```sh
echo "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJYTlpb0tsZlgv
N2t3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVG
dzB5TXpBNU1EWXdNekl3TkRsYUZ3MHpNekE1TURNd016STFORGxhTUJVeApFekFSQmdOVkJBTVRD
bXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJB
UUREeC83SDlucDd0VGdYWUdaV2RIWWxBazlXVUdMNjJBTlFXRTBZakl2TkdrMmNxR2tUOHBzdkF6
UkYKYzFkTHc4V3BURm41U054NG5lMU9sOVM3U1haenZoWkd1TURoSHZESVBWU0RjZGd6dWVacWlL
VGNQYWh6UE13egozaEVKMmd6WFF5NzRRL2tlVGQwcnF4T3NUMFRJUC9CRGlFVFFQejBLUW16bnRa
QWc1ZUVrRGg4MUk3aDE1OGp3Cm5iZXlJdGIybFdqRkpkSUpzNkUxT3NBajRCaDNXdWR6c0JCL2g1
MmFaN2NYcElRMDNPYkRLdzViQksrSUJCMzEKYW4raVVsVVI0eWtRU2l1S2hwVDBpT00wd1JvWGxK
Tkw3dUJ0Y2l3Nnlvc2s2ZTBJZHRReTlPYUovazhhMStveApyaDQxelQxenNnV0ZQZHY3Y1BXWCtI
UWhjSGZ2QWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJU
QURBUUgvTUIwR0ExVWREZ1FXQkJUVnNUN1cxVWZkY2Q5TVpVNDA4UU9XZUEvRWhUQVYKQmdOVkhS
RUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQUJ2ZmZ4UTFh
ZwpDc3pZUEtOZUhIMm9RWVhyYzJTKzNkNWltbEg2OFk0UlBkcW4vWW5MaEs2WE1IMTBsTXBBZ3BK
eE5qM1RWUGY1CjhRNk12MDhUanVGNkZISTlFYWFoUXBGUitoRFRXTlhQKzhkYzNsVTR2TWI4a2l5
YVJNQ0FhVEV1eFBFMzRYVlMKZWVvU0wxL0xUcGlhQUtVVWtIYXFUWnJibitHRkRMTE80dm1uRllt
Z0xiVkwweExUc09NOGQvYVlvM0lZK0g0bwpsaHE2WHJ5K2d1OVJITkpSWjdPRmVRSnlYaUFQRWpX
L1dESXZHUkw5NnRPYWk1TlZpMXFkRy90OEtBd2JkZXA3ClhyaUtQdEt0aTZNWWdYTmdDbGdyWlBG
ai9yL1RqNUpEWEJaVnR0RU5kUG1CcExKdWkxOTdVaHRQVER3VDZyWFcKak5GcVdmMytydjRUCi0t
LS0tRU5EIENFUlRJRklDQVRFLS0tLS0K" | base64 --decode
```