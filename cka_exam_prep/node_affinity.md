# [Node Affinity and Anti-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)

- Node affinity is conceptually similar to nodeSelector, allowing you to constrain which nodes your Pod can be scheduled on based on node labels. There are two types of node affinity:

  - **requiredDuringSchedulingIgnoredDuringExecution:** The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
  - **preferredDuringSchedulingIgnoredDuringExecution:** The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

- For example, consider the following Pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values:
                  - antarctica-east1
                  - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
              - key: another-node-label-key
                operator: In
                values:
                  - another-node-label-value
  containers:
    - name: with-node-affinity
      image: registry.k8s.io/pause:2.0
```

- In this example, the following rules apply:

  - The node must have a label with the key topology.kubernetes.io/zone and the value of that label must be either antarctica-east1 or antarctica-west1.
  - The node preferably has a label with the key another-node-label-key and the value another-node-label-value.

- You can use the operator field to specify a logical operator for Kubernetes to use when interpreting the rules. You can use In, NotIn, Exists, DoesNotExist, Gt and Lt.
  - **In:** The label value is present in the supplied set of strings
  - **NotIn:** The label value is not contained in the supplied set of strings
  - **Exists:** A label with this key exists on the object
  - **DoesNotExist:** No label with this key exists on the object
  - **Gt:** The supplied value will be parsed as an integer, and that integer is less than the integer that
    results from parsing the value of a label named by this selector
  - **Lt:** The supplied value will be parsed as an integer, and that integer is greater than the integer that results from parsing the value of a label named by this selector

### Node affinity weight

- You can specify a weight between 1 and 100 for each instance of the **preferredDuringSchedulingIgnoredDuringExecution** affinity type. When the scheduler finds nodes that meet all the other scheduling requirements of the Pod, the scheduler iterates through every preferred rule that the node satisfies and adds the value of the weight for that expression to a sum.

- The final sum is added to the score of other priority functions for the node. Nodes with the highest total score are prioritized when the scheduler makes a scheduling decision for the Pod.

- For example, consider the following Pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-affinity-anti-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                  - linux
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
              - key: label-1
                operator: In
                values:
                  - key-1
        - weight: 50
          preference:
            matchExpressions:
              - key: label-2
                operator: In
                values:
                  - key-2
  containers:
    - name: with-node-affinity
      image: registry.k8s.io/pause:2.0
```

- If there are two possible nodes that match the **preferredDuringSchedulingIgnoredDuringExecution** rule, one with the label-1:key-1 label and another with the label-2:key-2 label, the scheduler considers the weight of each node and adds the weight to the other scores for that node, and schedules the Pod onto the node with the highest final score.

## [Assign Pods to Nodes using Node Affinity](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)
