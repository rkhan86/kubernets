# [Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

## Requests and limits

If the node where a Pod is running has enough of a resource available, it's possible (and allowed) for a container to use more resource than its request for that resource specifies. However, a container is not allowed to use more than its resource limit.

## CPU Resources

In Kubernetes, CPU resources can be allocated to Pods and containers to ensure that they have the required processing power to run their applications. CPU resources are defined using the CPU resource units (millicores or milliCPU) and can be specified as CPU requests and CPU limits.

### CPU Requests and Limits with YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: my-image
      resources:
        requests:
          cpu: "100m"
        limits:
          cpu: "200m"
```

### Note:

    - Kubernetes doesn't allow you to specify CPU resources with a precision finer than 1m. Because of this, it's useful to specify CPU units less than 1.0 or 1000m using the milliCPU form; for example, 5m rather than 0.005.
    - 1 CPU means 1 AWS vCPU, 1 GCP Core, 1 Azure Core, 1 Hyperthread

### Impact of Exceeding CPU Limits

     If a container tries to exceed its CPU limit, it will be throttled by the kernel. This can cause the container to become unresponsive, leading to degraded performance and potentially impacting other containers running on the same node.

## Memory Resources

In Kubernetes, memory resources can be allocated to Pods and containers to ensure that they have the required memory to run their applications. Memory resources are defined using the memory resource units (such as bytes, kilobytes, megabytes, or gigabytes) and can be specified as memory requests and memory limits.

### Memory Requests and Limits with YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: my-image
      resources:
        requests:
          memory: "64Mi"
        limits:
          memory: "128Mi"
```

Limits and requests for memory are measured in bytes. You can express memory as a plain integer or as a fixed-point number using one of these quantity suffixes: E, P, T, G, M, k. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, Ki. For example, the following represent roughly the same value:

```
128974848, 129e6, 129M,  128974848000m, 123Mi
```

```
1 G (Gigabyte)  = 1,000,000.000 bytes
1 M (Megabyte)  = 1,000,000 bytes
1 K (Kilobyte)  = 1,000 bytes

1 Gi(Gibibyte) =  1,073,741,824 bytes
1 Mi(Mebibyte) =  1,048,576 bytes
1 Ki(Kibibyte) =  1,024 bytes
```

### Impact of Exceeding Memory Limits

    When a container exceeds its memory limit in Kubernetes, it is terminated and restarted by Kubernetes. This event is known as OOMKilled, which stands for Out Of Memory Killed.

## Configure Default Memory Requests and Limits for a Namespace

- Create a namespace so that the resources you create in this exercise are isolated from the rest of your cluster.

```sh
kubectl create namespace default-mem-example
```

- Create a LimitRange and a Pod

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
    - default:
        cpu: 1Gi # limites
      defaultRequest:
        cpu: 1Gi # request
      max:
        cpu: 1Gi # limites
      min:
        cpu: 500Mi #request
      type: Container
```

- Create the LimitRange in the default-mem-example namespace:

```sh
kubectl apply -f limit_range.yaml --namespace=default-mem-example
```

## Configure Default CPU Requests and Limits for a Namespace

- Create a namespace so that the resources you create in this exercise are isolated from the rest of your cluster.

```sh
kubectl create namespace default-mem-example
```

- Create a LimitRange and a Pod

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
    - default:
        cpu: 500m # limites
      defaultRequest:
        cpu: 256m # request
      max:
        cpu: "1" # limites
      min:
        cpu: 100m #request
      type: Container
```

- Create the LimitRange in the default-mem-example namespace:

```sh
kubectl apply -f limit_range.yaml --namespace=default-mem-example
```
