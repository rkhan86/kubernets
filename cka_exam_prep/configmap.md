# [Configmap](https://kubernetes.io/docs/concepts/configuration/configmap/)

- There are two phases involved in configuring config maps.First, create the config map, and second inject them into the pod.

## Create the config map:

- Just like any other Kubernetes object, there are two ways of creating a config map. The imperative way without using a config map definition file, and the declarative way, by using a config map definition file.

### The imperative way

- To create a config map of the given values, run the Kube control create config map command. The command is followed by the config name and the option from literal. The from literal option is used to specify the key value pairs in the command itself. If you wish to add additional key value pairs, simply specify the from literal options multiple times.

```sh
kubectl create configmap <config-name> --from-literal=<key1>=<value1> --from-literal=<key2>=<value2>
```

- Another way to input configuration data is through a file.Use the from file option to specify a path to the file that contains the required data.

```sh
kubectl create configmap <config-name> --from-file=/path/to/the/file
```

### The declarative way

- For this, we create a definition file, just like how we did for the pod.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: CM
  namespace: default
data:
  key: default
```

```sh
kubectl create -f file_name.yaml
kubectl get configmaps
kubectl describe configmaps
```

## Configuring it with a pod

- There are many ways to inject configuration data into pods. You can inject it as a single environment variable or, you can inject the whole data as files in a volume.

### Singel env

```yaml
env:
  - name: DB_HOST
    valueFrom:
      configMapKeyRef:
        name: CM
        key: DB_HOST
```

### Env variables

```yaml
envFrom:
  - configMapRef:
      name: CM
```

### Files in a volume

```yaml
volumes:
  - name: configmap
    configMap:
      name: MYAPP
```

- An example of POD yaml file with configmap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "MYAPP"
  namespace: default
  labels:
    app: "MYAPP"
spec:
  containers:
    - name: MYAPP
      image: "debian-slim:latest"
      ports:
        - containerPort: 80
          name: http
      envFrom:
        - configMapRef:
            name: MYAPP
```

- For more detailes:
  [**Configure a Pod to Use a ConfigMap**](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
