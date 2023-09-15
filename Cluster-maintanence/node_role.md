
   #  <u>Node Role</u>
    The label applied to control-plane nodes "node-role.kubernetes.io/master" is now deprecated and will be removed in a future release after a GA deprecation period.
    Introduce a new label "node-role.kubernetes.io/control-plane" that will be applied in parallel to "node-role.kubernetes.io/master" until the removal of the "node-role.kubernetes.io/master" label.

I think also important is this:

    Make "kubeadm upgrade apply" add the "node-role.kubernetes.io/control-plane" label on existing nodes that only have the "node-role.kubernetes.io/master" label during upgrade.
    Please adapt your tooling built on top of kubeadm to use the "node-role.kubernetes.io/control-plane" label.
    The taint applied to control-plane nodes "node-role.kubernetes.io/master:NoSchedule" is now deprecated and will be removed in a future release after a GA deprecation period.
    Apply toleration for a new, future taint "node-role.kubernetes.io/control-plane:NoSchedule" to the kubeadm CoreDNS / kube-dns managed manifests. Note that this taint is not yet applied to kubeadm control-plane nodes.
    Please adapt your workloads to tolerate the same future taint preemptively.



**node-role.kubernetes.io/control-plane**
```
Type: Label

Used on: Node

A marker label to indicate that the node is used to run control plane components. The kubeadm tool applies this label to the control plane nodes that it manages. Other cluster management tools typically also set this taint.

You can label control plane nodes with this label to make it easier to schedule Pods only onto these nodes, or to avoid running Pods on the control plane. If this label is set, the EndpointSlice controller ignores that node while calculating Topology Aware Hints.
```

**node-role.kubernetes.io/control-plane**
```
Type: Taint

Example: node-role.kubernetes.io/control-plane:NoSchedule

Used on: Node

Taint that kubeadm applies on control plane nodes to restrict placing Pods and allow only specific pods to schedule on them.

```
- If this Taint is applied, control plane nodes allow only critical workloads to be scheduled onto them. You can manually remove this taint with the following command on a specific node.
```sh
kubectl taint nodes <node-name> node-role.kubernetes.io/control-plane:NoSchedule-
```
- List all node
```sh
kubectl get nodes 
```
- Add role:
```sh
kubectl label node <node name> node-role.kubernetes.io/<role name>=
```
- Remove role:
```sh
kubectl label node <node name> node-role.kubernetes.io/<role name>-
```
**example:**
```sh
kubectl label nodes worker-2.example.com node-role.kubernetes.io/worker=
kubectl label node worker-2.example.com node-role.kubernetes.io/worker-
```