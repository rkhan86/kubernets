# [Static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)

- In Kubernetes, a static pod is a pod that is managed directly by the kubelet on a specific node, without going through the Kubernetes API server. Therefore, you cannot update the configuration of a static Pod using kubectl, and you cannot use features like rolling updates or resource quotas with static Pods.

- In Kubernetes, a staticPodPath is a configuration setting that specifies the directory path where the kubelet service running on a node should look for static pod manifests.

```sh
cat /var/lib/kubelet/config.yaml | grep -i staticPodPath
```

- Add pod definition file to staticPodPath

```sh
 kubectl run <static_pod_name> --image=<image> --dry-run=client -o yaml > <staticPodPath>/<static_pod_file_name>.yaml
```

- Edit the pod definition file

```sh
 vi <staticPodPath>/<static_pod_file_name>.yaml
```

- Remove the pod definition file from staticPodPath

```sh
 rm <staticPodPath>/<static_pod_file_name>.yaml
```

## No Static Deployment, Static ReplicaSet, or Static Service Objects

- Static Pods are the only type of “static” object in Kubernetes. The kubelet works at the Pod level because it is responsible for managing the lifecycle of each Pod running on the node.

## Static Pods vs. DaemonSets

- Static Pods and DaemonSets are both Kubernetes objects that are used to manage pods, but they serve different purposes.

| Features                      |              Static POD              |               Daemonsets                |
| :---------------------------- | :----------------------------------: | :-------------------------------------: |
| Functionalities in kubernetes | created and managed by specific node | POD is running in every node in cluster |
| Management                    |               kubelet                |  kube API server(Daemonset controller)  |
| Deployment scope              |            specific node             |           accross the cluster           |
| Kube-sheduler                 |              no effect               |                no effect                |
