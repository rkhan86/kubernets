# [Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

- **Taints** are set on Node and **Tolaration** are set on POD

- **Taints** allow a node to repel a set of pods.

- **Tolerations** are applied to pods. Tolerations allow the scheduler to schedule pods with matching taints. Tolerations allow scheduling but don't guarantee scheduling: the scheduler also evaluates other parameters as part of its function.

## Taints

- You add a taint to a node using kubectl taint. For example,

```sh
kubectl taint nodes node-name key=value:taint-effect
```

- To remove the taint added by the command above, you can run:

```sh
kubectl taint nodes node1 key1=value1:taint-effect-
```

### taint-effect: NoSchedule | PreferNoSchedule | NoExecute

- Taint effects define what will happen to pods if they don’t tolerate the taints. The three taint effects are:

  - NoSchedule: A strong effect where the system lets the pods already scheduled in the nodes run, but enforces taints from the subsequent pods.
  - PreferNoSchedule: A soft effect where the system will try to avoid placing a pod that does not tolerate the taint on the node.
  - NoExecute: A strong effect where all previously scheduled pods are evicted, and new pods that don’t tolerate the taint will not be scheduled.

- The three taint effects can be seen here:

```sh
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key2=value1:PreferNoSchedule
```

- The NoExecute taint effect, mentioned above, affects pods that are already running on the node as follows

  - pods that do not tolerate the taint are evicted immediately
  - pods that tolerate the taint without specifying **tolerationSeconds** in their toleration specification remain bound forever
  - pods that tolerate the taint with a specified **tolerationSeconds** remain bound for the specified amount of time

- The node controller automatically taints a Node when certain conditions are true. The following taints are built in:

  - node.kubernetes.io/not-ready: Node is not ready. This corresponds to the NodeCondition Ready being "False".
  - node.kubernetes.io/unreachable: Node is unreachable from the node controller. This corresponds to the - NodeCondition Ready being "Unknown".
  - node.kubernetes.io/memory-pressure: Node has memory pressure.
  - node.kubernetes.io/disk-pressure: Node has disk pressure.
  - node.kubernetes.io/pid-pressure: Node has PID pressure.
  - node.kubernetes.io/network-unavailable: Node's network is unavailable.
  - node.kubernetes.io/unschedulable: Node is unschedulable.
  - node.cloudprovider.kubernetes.io/uninitialized: When the kubelet is started with "external" cloud provider, this taint is set on a node to mark it as unusable. After a controller from the cloud-controller-manager initializes this node, the kubelet removes this taint.

## Tolerations

- You specify a toleration for a pod in the PodSpec. Both of the following tolerations "match" the taint created by the kubectl taint line below, and thus a pod with either toleration would be able to schedule onto node1:

```sh
kubectl taint nodes node1 key1=value1:NoSchedule
```

```yaml
tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
```

```yaml
tolerations:
  - key: "key1"
    operator: "Exists"
    effect: "NoSchedule"
```

- Here's an example of a pod that uses tolerations:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
  tolerations:
    - key: "example-key"
      operator: "Exists"
      effect: "NoSchedule"
```

### Operator

- The default value for operator is Equal.

- A toleration "matches" a taint if the keys are the same and the effects are the same, and:

  - the operator is Exists (in which case no value should be specified), or
  - the operator is Equal and the values are equal.

**Note:**

- There are two special cases:

  - An empty key with operator Exists matches all keys, values and effects which means this will tolerate everything.
  - An empty effect matches all effects with key key1

- You can specify tolerationSeconds for a Pod to define how long that Pod stays bound to a failing or unresponsive Node.

```yaml
tolerations:
  - key: "node.kubernetes.io/unreachable"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 6000
```

### [Well-Known Labels, Annotations and Taints](https://kubernetes.io/docs/reference/labels-annotations-taints/)
