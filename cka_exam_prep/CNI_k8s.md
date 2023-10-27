## <img title="CNI" src="https://raw.githubusercontent.com/containernetworking/cni/main/logo.png" alt="CNI-logo" style="height: 30px; width:30px;"/> Introduction

The [Container Network Interface(CNI)](https://github.com/containernetworking/cni) is a set of standards that define how programs should be developed to solve networking challenges in a container runtime environments. The programs are referred to as plugins.

CNI defines how the plugin should be developed and how container run times should invoke them. CNI defines a set of responsibilities for container run times and plugins. For container run times, CNI specifies that it is responsible for creating a network name space for each container.

CNI defines a set of responsibilities for container run times and plugins. They are:

- Container Runtime must create network namespace
- Identify network the container must attach to
- Container Runtime to invoke Network Plugin (bridge) when container is ADDed.
- Container Runtime to invoke Network Plugin (bridge) when container is DELeted.
- JSON format of the Network Configuration

- Must support command line arguments ADD/DEL/CHECK
- Must support parameters container id, network ns etc..
- Must manage IP Address assignment to PODs
- Must Return results in a specific format

## Networking Cluster Nodes

|         Service         |          Port           |
| :---------------------: | :---------------------: |
|          ETCD           | 2379,2380(ETCD_Cluster) |
|         kubelet         |          10251          |
|        kube-api         |          6443           |
|     Kube-scheduler      |          10251          |
| Kube-controller-manager |          10252          |

```sh
kubectl get nodes -o wide
# The interface/bridge created by containerd on host
ip address show type bridge
# Default gateway
ip route
# Port that Kube-scheduler is using
netstat -npl | grep -i scheduler
# Port that are using by ETCD
netstat -npl | grep -i etcd
# Number of connection each port of etcd has
netstat -npa | grep -i etcd | grep -i 2379 | wc -l
netstat -npa | grep -i etcd | grep -i 2380 | wc -l
```

## Pod networking in Kubernetes

As of today, Kubernetes does not come with a built-in solution for this.However, Kubernetes have laid out clearly
the requirements for Pod networking.

### Networking Model

- Every POD should have an IP Address.
- Every POD should be able to communicate with every other POD in the same node.
- Every POD should be able to communicate with every other POD on other nodes without NAT.

## CNI in Kubernetes.

- from /etc/containerd/config.toml

```toml
[plugins."io.containerd.grpc.v1.cri".cni]
     bin_dir = "/opt/cni/bin"
     conf_dir = "/etc/cni/net.d"
```

- For Calico CNI Installed with tiger operator

**--network-plugin=cni** --> Network plugin set to CNI

**--cni-bin-dir=/opt/cni/bin** --> The CNI bin directory has all the supported CNA plugins as executables,such as the bridge, dscp, flannel, et cetera.</br>

**--cni-conf-dir=/etc/cni/net.d** --> The CNI conflist directory has a set of configuration files. This is where Cube Lead looks to find out which plugin needs to be used.</br>

- In /etc/cni/net.d directory calico create a script 10-calico.conflist, which is like that

```conflist
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "calico",
      "datastore_type": "kubernetes",
      "mtu": 0,
      "nodename_file_optional": false,
      "log_level": "Info",
      "log_file_path": "/var/log/calico/cni/cni.log",
      "ipam": { "type": "calico-ipam", "assign_ipv4" : "true", "assign_ipv6" : "false"},
      "container_settings": {
          "allow_ip_forwarding": false
      },
      "policy": {
          "type": "k8s"
      },
      "kubernetes": {
          "k8s_api_root":"https://10.96.0.1:443",
          "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "bandwidth",
      "capabilities": {"bandwidth": true}
    },
    {"type": "portmap", "snat": true, "capabilities": {"portMappings": true}}
  ]
}
```

- The CNI conflist directory has a set of configuration files. This is where Cube Lead looks to find out which plugin needs to be used. If there are multiple files here it will choose the one in alphabetical order.
- The IPAM section defines IPAM configuration. This is where you specify the subnet or the range of IP addresses that will be assigned to pods and any necessary roads.

```
 "ipam": { "type": "calico-ipam", "assign_ipv4" : "true", "assign_ipv6" : "false"},
```

## IP address management (IPAM)

- IPAM is how are the virtual bridge networks in the nodes assigned an IP subnet and how are the pods assigned an IP.

```conflist
"ipam": { "type": "calico-ipam", "assign_ipv4" : "true", "assign_ipv6" : "false"},
```

- When we initiate Kubeadm we provide pod network cidr (--pod-network-cidr) as follows:

```sh
kubeadm init --pod-network-cidr 10.10.18.0/16 --kubernetes-version 1.26.8 --node-name k8s-master
```

- Then we Install CNI Calico and provide the same --pod-network-cidr as follows

```yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    # Note: The ipPools section cannot be modified post-install.
    ipPools:
      - blockSize: 26
        cidr: 10.244.0.0/16 # This is --pod-network-cidr
        encapsulation: VXLANCrossSubnet
        natOutgoing: Enabled
        nodeSelector: all()
```

- From given --pod-network-cidr, IPAM get the network address and divided into subnet and then assign them for each nodes pod network. For example if I provide --pod-network-cidr 10.244.0.0/16 then IPAM will create some sub-net like 10.244.1.0/24, 10.244.2.0/24, 10.244.3.0/24 and will assign them to each nodes POD network, then every POD on that nodes will get the ip address from assign subnet address by IPAM.

## Service Networking

- While a pod is hosted on a node, a service is hosted across the cluster. It is not bound to a specific node,but remember, the service is only accessible from within the cluster. This type of service is known as ClusterIP.
- To make the application on the pod accessible outside the cluster, we create another service of type NodePort.This service also gets an IP address assigned to it and works just like ClusterIP, as in all the other pods can access this service using it's IP. But in addition, it also exposes the application on a port on all nodes in the cluster. That way external users or applications have access to the service.
- Our focus is:

  - How are the services getting these IP addresses?
  - how are they made available across all the nodes in the cluster?
  - How is the service made available to external users through a port on each node?
  - Who is doing that and how and where do we see it?

**For PODs**

- Each kubelet service on each node watches the changes in the cluster through the Kube API server, and every time a new pod is to be created, it creates the pod on the nodes. It then invokes the CNI plugin to configure networking for that pod.

**For Services**

- Similarly, each node runs another component known as kube-proxy. kube-proxy watches the changes in the cluster through Kube API server, and every time a new service is to be created, kube-proxy gets into action.Unlike pods, services are not created on each node or assigned to each node. Services are a cluster-wide concept. They exist across all the nodes in the cluster. As a matter of fact, they don't exist at all.

- There is no server or service really listening on the IP of the service. We have seen that pods have containers,and containers have namespaces with interfaces, and IPs assigned to those interfaces. With services, nothing like that exists. There are no processes or namespaces or interfaces for a service. It's just a virtual object. Then how do they get an IP address, and how are we able to access the application on the pod through a service?

- When we create a service object in Kubernetes, it is assigned an IP address from a predefined range. The kube-proxy components running on each node gets that IP address and creates forwarding rules on each node in the cluster. Saying, "Any traffic coming to this IP, the IP of the service, should go to the IP of the pod." Once that is in place, whenever a pod tries to reach the IP of the service, it is forwarded to the pod's IP address,which is accessible from any node in the cluster. Now, remember, it's not just the IP,it's an IP and port combination. Whenever services are created or deleted, the kube-proxy component creates or deletes these rules.

- kube-proxy supports different ways, such as userspace where kube-proxy listens on a port for each service and proxy's connections to the pods by creating IPVS rules.Or the third and the default option,and the one familiar to us is using iptables.The proxy mode can be set using the proxy mode option,while configuring the kube-proxy service.If this is not set, it defaults to iptables.

```yaml
kube-proxy --proxy-mode [userspace | iptables | ipvs ] …
```

- This range of cluster IP is specified in the Kube API server's option called service cluster IP range,you can find out it by below command

```sh
ps aux | grep kube-apiserver | grep -i service-cluster-ip-range
```

- out put will be like this

```
root        1248 13.8  6.9 1170516 421872 ?      Ssl  18:53   2:46 kube-apiserver --advertise-address=192.168.50.131 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key

```

**<span style="color: red;">Note:</span>** </br>

```
 --service-cluster-ip-range=10.96.0.0/12 is default cluster ip for k8s cluster. We also apply --pod-network-cidr in k8s cluster for POD network.For each of these networks, it shouldn't overlap. There shouldn't be a case where a pod,and a service are assigned the same IP address.
```

- You can see the rules created by kube-proxy in the IP table's NAT table output. Search for the name of the service as all rules created by kube-proxy have a comment with the name of the service on it.

```sh
iptables –L –t net | grep <service_name>
```

- Similarly, when you create a services of type NodePort, kube-proxy creates IP table rules to forward all traffic coming on a port on all nodes to the respective backend parts.You can also see kube-proxy create these entries in the kube-proxy logs itself. In the logs, you will find what proxier it uses. In this case, it's iptables, and then adds an entry when it added a new service for the app. Note that the location of this file might vary depending on your installation

```sh
tail -f /var/log/containers/kube-proxy-n2s6w_kube-system_kube-proxy-7d94ed8c22b562d65143c8c337f4fcba38d18a51a017efe78cb0895158502491.log
```

- output

```
2023-10-10T18:53:54.068594347Z stderr F I1010 18:53:54.068548       1 conntrack.go:100] "Set sysctl" entry="net/netfilter/nf_conntrack_tcp_timeout_close_wait" value=3600
2023-10-10T18:53:54.073501662Z stderr F I1010 18:53:54.071585       1 config.go:188] "Starting service config controller"
2023-10-10T18:53:54.073543377Z stderr F I1010 18:53:54.071626       1 shared_informer.go:311] Waiting for caches to sync for service config
2023-10-10T18:53:54.073549663Z stderr F I1010 18:53:54.072174       1 config.go:97] "Starting endpoint slice config controller"
2023-10-10T18:53:54.073592321Z stderr F I1010 18:53:54.072186       1 shared_informer.go:311] Waiting for caches to sync for endpoint slice config
2023-10-10T18:53:54.073601292Z stderr F I1010 18:53:54.072928       1 config.go:315] "Starting node config controller"
2023-10-10T18:53:54.073606891Z stderr F I1010 18:53:54.072942       1 shared_informer.go:311] Waiting for caches to sync for node config
2023-10-10T18:53:54.173478832Z stderr F I1010 18:53:54.173214       1 shared_informer.go:318] Caches are synced for node config
2023-10-10T18:53:54.173505796Z stderr F I1010 18:53:54.173274       1 shared_informer.go:318] Caches are synced for endpoint slice config
2023-10-10T18:53:54.173512385Z stderr F I1010 18:53:54.173288       1 shared_informer.go:318] Caches are synced for service config

```

## DNS in the Kubernetes cluster

- In this Section, we will see what names are assigned to what objects, what are service DNS records, pod DNS records, what are the different ways you can reach one pod from another.

- Kubernetes deploys a built-in DNS server by default when you set up a cluster. This DNS only work within the cluster.

- Whenever a service is created, the Kubernetes DNS service,creates a record for the service. It maps the service name to the IP address, so, within the cluster, any pod can now reach this service using its service name. Remember we talked about name spaces earlier, that everyone within the name space address each other just with their first names, and to address anyone in another name space, you use their full names.

- Let's assume the web service was in a separate namespace named Apps. Then to refer to it from the default namespace you would have to say Web Service. apps. The last name of the service, is now the name of the namespace.So here, Web Service is the name of the service,and Apps, is the name of the name space.

- For each name space the DNS server creates a Sub domain. All the services are grouped together into another Sub Domain called SVC.

### DNS for Service

- Web service is the name of the service and Apps is the name of the name space. For each name space, the DNS server creates a Sub domain with its name. All pods and services for a name space are thus grouped together within a Sub domain in the name of the name space. All the services are further grouped together into another Sub domain called SVC. So you can reach your application, with the name webservice.apps.svc. Finally, all the services and pods are grouped together, into a route domain for the cluster, which is set to cluster.local by default. So you can access the service using the URL webservice.apps.svc.cluster.local and that's the fully qualified domain name for the service. So that's how services are resolved within the cluster.

|  Hostname   | Namespace | Type |     Root      |  IP Address   |
| :---------: | :-------: | :--: | :-----------: | :-----------: |
| web-service |   apps    | svc  | cluster.local | 10.107.37.188 |

```sh
curl http://web-service.apps.svc.cluster.local
```

### DNS for PODs

- For each pod, Kubernetes generates a name by replacing the dots in the IP address with dashes. The name space remains the same and type is set to pod.

The route domain is always cluster.local

|  Hostname  | Namespace | Type |     Root      | IP Address |
| :--------: | :-------: | :--: | :-----------: | :--------: |
| 10-244-2-5 |   apps    | pod  | cluster.local | 10.244.2.5 |
| 10-244-1-5 |  default  | pod  | cluster.local | 10.244.1.5 |

```sh
curl http://10-244-2-5.apps.pod.cluster.local
```

### Core DNS in Kubernetes
