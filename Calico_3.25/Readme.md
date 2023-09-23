# _Installing Calico_v3.25 as Container Network Interface (CNI) plugins for cluster networking(Only in Master node)_

- [Quickstart_Guide](https://docs.tigera.io/calico/3.25/getting-started/kubernetes/quickstart)

```sh
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml
wget https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml
vim custom-resources.yaml <<<<<< match CIDR wirh --pod-network-cidr for pods if its custom
kubectl apply -f custom-resources.yaml
watch kubectl get pods -n calico-system
```

- To see the Calico API server pod become ready, and Calico API resources become available. You can check whether the APIs are available with the following command

```sh
kubectl api-resources | grep '\sprojectcalico.org'
```

- You should see the following output:

```
NAME                              SHORTNAMES                                      APIVERSION                 NAMESPACED   KIND
bgpconfigurations                 bgpconfig,bgpconfigs                            projectcalico.org/v3       false        BGPConfiguration
bgppeers                                                                          projectcalico.org/v3       false        BGPPeer
blockaffinities                   blockaffinity,affinity,affinities               projectcalico.org/v3       false        BlockAffinity
caliconodestatuses                caliconodestatus                                projectcalico.org/v3       false        CalicoNodeStatus
clusterinformations               clusterinfo                                     projectcalico.org/v3       false        ClusterInformation
felixconfigurations               felixconfig,felixconfigs                        projectcalico.org/v3       false        FelixConfiguration
globalnetworkpolicies             gnp,cgnp,calicoglobalnetworkpolicies            projectcalico.org/v3       false        GlobalNetworkPolicy
globalnetworksets                                                                 projectcalico.org/v3       false        GlobalNetworkSet
hostendpoints                     hep,heps                                        projectcalico.org/v3       false        HostEndpoint
ipamconfigurations                ipamconfig                                      projectcalico.org/v3       false        IPAMConfiguration
ippools                                                                           projectcalico.org/v3       false        IPPool
ipreservations                                                                    projectcalico.org/v3       false        IPReservation
kubecontrollersconfigurations                                                     projectcalico.org/v3       false        KubeControllersConfiguration
networkpolicies                   cnp,caliconetworkpolicy,caliconetworkpolicies   projectcalico.org/v3       true         NetworkPolicy
networksets                       netsets                                         projectcalico.org/v3       true         NetworkSet
profiles                                                                          projectcalico.org/v3       false        Profile
```

## <u>Install calicoctl as a kubectl plugin on a single host</u>

```sh
curl -L https://github.com/projectcalico/calico/releases/download/v3.25.0/calicoctl-linux-amd64 -o kubectl-calico
mv kubectl-calico /usr/local/sbin/
chmod 755 /usr/local/sbin/kubectl-calico
```

- Usage:

```
kubectl calico -h
kubectl calico node status
kubectl calico ipam show
kubectl-calico get ipPool
kubectl-calico get profile
kubectl-calico get kubeControllersConfiguration -o yaml
kubectl-calico ipam show --show-blocks
kubectl-calico ipam show --show-borrowed
kubectl-calico ipam show --show-configuration
```

- [**Upgrade Calico on Kubernetes**](https://docs.tigera.io/calico/latest/operations/upgrading/kubernetes-upgrade)
- [**Upgrading an installation that uses manifests and the Kubernetes API datastore**](https://docs.tigera.io/calico/latest/operations/upgrading/kubernetes-upgrade#upgrading-an-installation-that-uses-manifests-and-the-kubernetes-api-datastore)