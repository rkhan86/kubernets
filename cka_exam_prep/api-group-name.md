```sh
curl https://localhost:6443/apis -k  --key admin.key --cert admin.crt --cacert /etc/kubernetes/pki/ca.crt | grep "name"
```

```
     "name": "apiregistration.k8s.io",
     "name": "apps",
     "name": "events.k8s.io",
     "name": "authentication.k8s.io",
     "name": "authorization.k8s.io",
     "name": "autoscaling",
     "name": "batch",
     "name": "certificates.k8s.io",
     "name": "networking.k8s.io",
     "name": "policy",
     "name": "rbac.authorization.k8s.io",
     "name": "storage.k8s.io",
     "name": "admissionregistration.k8s.io",
     "name": "apiextensions.k8s.io",
     "name": "scheduling.k8s.io",
     "name": "coordination.k8s.io",
     "name": "node.k8s.io",
     "name": "discovery.k8s.io",
     "name": "flowcontrol.apiserver.k8s.io",
     "name": "projectcalico.org",
     "name": "crd.projectcalico.org",
     "name": "operator.tigera.io",
```

```
curl https://192.168.50.151:6443/apis -k  --key admin151.key --cert admin151.crt --cacert ca.crt | grep "name"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0      "name": "apiregistration.k8s.io",
      "name": "apps",
      "name": "events.k8s.io",
      "name": "authentication.k8s.io",
      "name": "authorization.k8s.io",
      "name": "autoscaling",
      "name": "batch",
      "name": "certificates.k8s.io",
      "name": "networking.k8s.io",
      "name": "policy",
      "name": "rbac.authorization.k8s.io",
      "name": "storage.k8s.io",
      "name": "admissionregistration.k8s.io",
      "name": "apiextensions.k8s.io",
      "name": "scheduling.k8s.io",
100  6765    0  6765    0     0   151k      0 --:--:-- --:--:-- --:--:--  153k
      "name": "coordination.k8s.io",
      "name": "node.k8s.io",
      "name": "discovery.k8s.io",
      "name": "flowcontrol.apiserver.k8s.io",
      "name": "projectcalico.org",
      "name": "crd.projectcalico.org",
      "name": "operator.tigera.io",
      "name": "metrics.k8s.io",
```
