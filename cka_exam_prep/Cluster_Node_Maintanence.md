<u>**_Cluster Node Maintanence_**</u>

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
