# <u>_Installation_</u>

**Preparation**

- If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.Note, you don’t need this if you’re using kube-router as service-proxy because it is enabling strict ARP by default.

```sh
ks -n kube-system get all -o wide
ks -n kube-system get cm -o wide
ks -n kube-system describe cm kube-proxy
```
**If (mode: "ipvs") then set (strictARP: true**)
```yaml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```
- To edit it use 
```sh
ks -n kube-system edit cm kube-proxy
```

 **Installation by manifest**
- To install MetalLB, apply the manifest:
```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
wget https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
kubectl apply -f metallb.yaml
ks -n metallb-system get all -o wide
kubectl get pods -n metallb-system
```
- Delete Pod security policy (kind: PodSecurityPolicy) from metallb.yaml as it is depricated from kubernetes v1.25

- This will deploy MetalLB to your cluster, under the metallb-system namespace. The components in the manifest are:

    - The metallb-system/controller deployment. This is the cluster-wide controller that handles IP address assignments.
    - The metallb-system/speaker daemonset. This is the component that speaks the protocol(s) of your choice to make the services reachable.
    - Service accounts for the controller and speaker, along with the RBAC permissions that the components need to function.

- The installation manifest does not include a configuration file. MetalLB’s components will still start, but will remain idle until you [start deploying resources](https://metallb.universe.tf/configuration/).

**Layer 2 configuration**
- MetalLB remains idle until configured. This is accomplished by creating and deploying various resources into the same namespace (metallb-system) MetalLB is deployed into.
- By deploying it, the controller allocates IP addresses for the load-balancer. The IP address pool allows for one or many IP addresses, depending on what we need. The allocated addresses must be part of the host network of our cluster and unique.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - {ip-start}-{ip-stop}
```