# _Installing Calico_v3.25 as Container Network Interface (CNI) plugins for cluster networking(Only in Master node)_
- [Quickstart_Guide](https://docs.tigera.io/calico/3.25/getting-started/kubernetes/quickstart)
```sh
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml
wget https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml
vim custom-resources.yaml <<<<<< match CIDR wirh --pod-network-cidr for pods if its custom
kubectl apply -f custom-resources.yaml
watch kubectl get pods -n calico-system
```