# <u>**_Cluster Node Maintanence_**</u>

**<u>Cordon</u>**

- Kubernetes cordon is an operation that marks or taints a node in your existing node pool as unschedulable.The command prevents the Kubernetes scheduler from placing new pods onto that node, but it doesn’t affect existing pods on that node.
- To mark a node unschedulable, all it takes is running this command:

```sh
kubectl cordon <node name>
kubectl get nodes
```

- Note that you can run pods that are part of a DaemonSet on an unschedulable Node. That’s because DaemonSets usually provide node-local services that should be running on the node, even if it’s marked as unschedulable and drained of workloads.

- Once you've completed your maintenance, you can power the Node back up to reconnect it to your cluster. You must then remove the cordon you created to mark the Node as schedulable again:

```sh
kubectl uncordon <node name>
kubectl get nodes
```

---

**<u>Safely Drain a Node</u>**

- You can use kubectl drain to safely evict all of your pods from a node before you perform maintenance on the node (e.g. kernel upgrade, hardware maintenance, etc.). Safe evictions allow the pod's containers to gracefully terminate and will respect the PodDisruptionBudgets you have specified.

- When kubectl drain returns successfully, that indicates that all of the pods (except the ones excluded as described in the previous paragraph) have been safely evicted (respecting the desired graceful termination period, and respecting the PodDisruptionBudget you have defined). It is then safe to bring down the node by powering down its physical machine or, if running on a cloud platform, deleting its virtual machine.

- First, identify the name of the node you wish to drain. You can list all of the nodes in your cluster with

```sh
kubectl get nodes
```

- Next, tell Kubernetes to drain the node:

```sh
kubectl drain --ignore-daemonsets <node name>
```

- Also you can use "**--force --grace-period 0**" option. **Not recommended**.

```sh
kubectl drain --ignore-daemonsets <node name> --force --grace-period 0
```

**Overriding Pod Disruption Budgets**

- Pod disruption budgets are a mechanism that provide protection for your workloads. They shouldn't be overridden unless you must immediately shutdown a Node. The drain command's --disable-eviction flag provides a way to achieve this.

```sh
kubectl drain <node name> --disable-evictio
```

- This circumvents the regular Pod eviction process. Pods will be directly deleted instead, ignoring any applied disruption budgets.

- Once it returns (without giving an error), you can power down the node (or equivalently, if on a cloud platform, delete the virtual machine backing the node). If you leave the node in the cluster during the maintenance operation, you need to run

```sh
kubectl uncordon $NODENAME
kubectl get nodes
```

---

**<u>Delete a Node from cluster</u>**

- To begin working with nodes, you must first create a list of them. You may use the kubectl get nodes command to acquire a list of nodes. According to the command output, we have two nodes that are in the unknown and ready status:

```sh
kubectl get nodes
```

- To delete a specific node, you have drain it drain it by following above section and then use the following command is used:

```sh
kubectl delete node <node name>
```

- If you are using kubeadm and would like to reset the deleted Node to a state which was there before running kubeadm join then run from the node

```sh
kubeadm reset
```

---

**<u>Join a Node to cluster</u>**

- Check if you have a token – Run the command on Control node:

```sh
kubeadm token list
```

- Now You can also generate token and print the join command:

```sh
kubeadm token create --print-join-command
```

- Use the command to join the node

---

## **<u>(Optional) Configure a disruption budget</u>**

Ensuring the disruptions to your applications do not affect its availability isn't a simple task. The release of Kubernetes v1.26 lets you specify an unhealthy pod eviction policy for PodDisruptionBudgets (PDBs) to help you maintain that availability during node management operations. In this article, we will dive deeper into what modifications were introduced for PDBs to give application owners greater flexibility in managing disruptions.

**What problems does this solve?**

API-initiated eviction of pods respects PodDisruptionBudgets (PDBs). This means that a requested voluntary disruption via an eviction to a Pod, should not disrupt a guarded application and .status.currentHealthy of a PDB should not fall below .status.desiredHealthy. Running pods that are Unhealthy do not count towards the PDB status, but eviction of these is only possible in case the application is not disrupted. This helps disrupted or not yet started application to achieve availability as soon as possible without additional downtime that would be caused by evictions.

Unfortunately, this poses a problem for cluster administrators that would like to drain nodes without any manual interventions. Misbehaving applications with pods in CrashLoopBackOff state (due to a bug or misconfiguration) or pods that are simply failing to become ready make this task much harder. Any eviction request will fail due to violation of a PDB, when all pods of an application are unhealthy. Draining of a node cannot make any progress in that case.

On the other hand there are users that depend on the existing behavior, in order to:

    - prevent data-loss that would be caused by deleting pods that are guarding an underlying resource or storage
    - achieve the best availability possible for their application

Kubernetes 1.26 introduced a new experimental field to the PodDisruptionBudget API: .spec.unhealthyPodEvictionPolicy. When enabled, this field lets you support both of those requirements.

This is an alpha feature, which means you have to enable the PDBUnhealthyPodEvictionPolicy feature gate, with the command line argument --feature-gates=PDBUnhealthyPodEvictionPolicy=true to the kube-apiserver.

**Step 1: Enabling PDBUnhealthyPodEvictionPolicy feature gate in kube-apiserver for Kubernetes 1.26**

- Make a copy of /etc/kubernetes/manifests/kube-apiserver.yaml before you make changes to it.
- Make the change at kube-apiserver.yaml and save it.

```yaml

---
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.50.131:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
    - command:
        - kube-apiserver
        - --advertise-address=192.168.50.131
        - --allow-privileged=true
        - --authorization-mode=Node,RBAC
        - --client-ca-file=/etc/kubernetes/pki/ca.crt
        - --enable-admission-plugins=NodeRestriction
        - --enable-bootstrap-token-auth=true
        - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
        - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
        - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
        - --etcd-servers=https://127.0.0.1:2379
        - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
        - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
        - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
        - --requestheader-allowed-names=front-proxy-client
        - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
        - --requestheader-extra-headers-prefix=X-Remote-Extra-
        - --requestheader-group-headers=X-Remote-Group
        - --requestheader-username-headers=X-Remote-User
        - --secure-port=6443
        - --service-account-issuer=https://kubernetes.default.svc.cluster.local
        - --service-account-key-file=/etc/kubernetes/pki/sa.pub
        - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
        - --service-cluster-ip-range=10.96.0.0/12
        - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
        - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
        - --feature-gates=PDBUnhealthyPodEvictionPolicy=true # SET THIS LINE
      image: registry.k8s.io/kube-apiserver:v1.26.8
      imagePullPolicy: IfNotPresent
```

- Monitor containers with watch crictl ps or watch docker ps depending on the container runtime that the cluster uses.

```sh
watch crictl ps
```

- After successfully start the kube-apiserver container run below command to confirm it

```sh
ps aux | grep kube-apiserver | grep PDBUnhealthyPodEvictionPolicy
```

**Step 2: Object for Pod Disruption Budget**

- Now create a policy for pod-disruption-budget, for that you can follow this [Link](https://github.com/HariSekhon/Kubernetes-configs/blob/master/pod-disruption-budget.yaml) for yaml file

```sh
vim pod-disruption-budget.yaml
```

```yaml
#apiVersion: policy/v1beta1  # Kubernetes v1.5+
apiVersion: policy/v1 # Kubernetes v1.21+
kind: PodDisruptionBudget
metadata:
  name: zk-pdb
  namespace: NAMESPACE
spec:
  # minAvailable and maxUnavailable are mutually exclusive - both round up, so minAvailable is safer as maxUnavailable may round up and exceed your expectations
  #minAvailable: 4      # < 1.7  - must calculate yourself, eg 2/3, or 4/5
  minAvailable: 51% # XXX: Safer than maxUnavailable due to rounding up to more pod replicas
  #maxUnavailable: 25%  # 1.7+ - XXX: do not set to integers, you may cause outages if replicas <= maxUnavailable, and may exceed percentages due to rounding up replicas to take down
  selector:
    matchLabels:
      app: zookeeper
  #unhealthyPodEvictionPolicy: AlwaysAllow #This field is alpha-level. The eviction API uses this field when the feature gate PDBUnhealthyPodEvictionPolicy is enabled (disabled by default).
```

```sh
kubectl apply -f pod-disruption-budget.yaml
kubectl get poddisruptionbudgets
```

**Step 3: Run a POD**

- Use the **selector** from pod-disruption-budget.yaml in your pod deployment yaml.

**For more details follow the [YouTube_Link](https://www.youtube.com/watch?v=L1nCLcX5IAk)**

```
KIND:     PodDisruptionBudget
VERSION:  policy/v1

DESCRIPTION:
     PodDisruptionBudget is an object to define the max disruption that can be
     caused to a collection of pods

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object>
     Specification of the desired behavior of the PodDisruptionBudget.

   status	<Object>
     Most recently observed status of the PodDisruptionBudget.

------------------------------------------------------------------------------------------
KIND:     PodDisruptionBudget
VERSION:  policy/v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the PodDisruptionBudget.

     PodDisruptionBudgetSpec is a description of a PodDisruptionBudget.

FIELDS:
   maxUnavailable	<string>
     An eviction is allowed if at most "maxUnavailable" pods selected by
     "selector" are unavailable after the eviction, i.e. even in absence of the
     evicted pod. For example, one can prevent all voluntary evictions by
     specifying 0. This is a mutually exclusive setting with "minAvailable".

   minAvailable	<string>
     An eviction is allowed if at least "minAvailable" pods selected by
     "selector" will still be available after the eviction, i.e. even in the
     absence of the evicted pod. So for example you can prevent all voluntary
     evictions by specifying "100%".

   selector	<Object>
     Label query over pods whose evictions are managed by the disruption budget.
     A null selector will match no pods, while an empty ({}) selector will
     select all pods within the namespace.

   unhealthyPodEvictionPolicy	<string>
     UnhealthyPodEvictionPolicy defines the criteria for when unhealthy pods
     should be considered for eviction. Current implementation considers healthy
     pods, as pods that have status.conditions item with
     type="Ready",status="True".

     Valid policies are IfHealthyBudget and AlwaysAllow. If no policy is
     specified, the default behavior will be used, which corresponds to the
     IfHealthyBudget policy.

     IfHealthyBudget policy means that running pods (status.phase="Running"),
     but not yet healthy can be evicted only if the guarded application is not
     disrupted (status.currentHealthy is at least equal to
     status.desiredHealthy). Healthy pods will be subject to the PDB for
     eviction.

     AlwaysAllow policy means that all running pods (status.phase="Running"),
     but not yet healthy are considered disrupted and can be evicted regardless
     of whether the criteria in a PDB is met. This means perspective running
     pods of a disrupted application might not get a chance to become healthy.
     Healthy pods will be subject to the PDB for eviction.

     Additional policies may be added in the future. Clients making eviction
     decisions should disallow eviction of unhealthy pods if they encounter an
     unrecognized policy in this field.

     This field is alpha-level. The eviction API uses this field when the
     feature gate PDBUnhealthyPodEvictionPolicy is enabled (disabled by
     default).
```
