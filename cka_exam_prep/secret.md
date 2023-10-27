# [Secret](https://kubernetes.io/docs/concepts/configuration/secret/)

- Secrets are used to store sensitive information like passwords or keys. They're similar to ConfigMaps except that they're stored in an encoded format.

- there are two steps involved in working with secrets. First, create the secret and second, inject it into pod.

## Create the secret:

- There are two ways of creating a secret, the imperative way, without using a secret definition file. And the declarative way, by using a secret definition file.

### The imperative way

- To create a secret of the given values, run the Kube control create secret command. The command is followed by the secret name and the option from literal. The from literal option is used to specify the key value pairs in the command itself. If you wish to add additional key value pairs, simply specify the from literal options multiple times.

```sh
kubectl create secret <secret-name> --from-literal=<key1>=<value1> --from-literal=<key2>=<value2>
```

- Another way to input configuration data is through a file.Use the from file option to specify a path to the file that contains the required data.

```sh
kubectl create secret <secret-name> --from-file=/path/to/the/file
```

### The declarative way

- For this, we create a definition file, just like how we did for the pod.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: default
type: Opaque
data:
  # Example:
  # password: {{ .Values.password | b64enc }}
  DB_Host: blcVfg$E
  DB_User: ght#gh=J
  DB_Password: jklY0!g
```

- But how do you convert the data from plain text to an encoded format

```sh
echo -n 'password' | base64 -w 0
```

```sh
kubectl create -f file_name.yaml
kubectl get secrets
kubectl describe secrets
kubectl get secret <secret_name> -o yaml
```

- Now, how do you decode encoded values

```sh
echo -n 'jklY0!g' | base64 --decode
```

## Configuring it with a pod

- There are many ways to inject secret data into pods. You can inject it as a single environment variable or, you can inject the whole data as files in a volume.

### Singel env

```yaml
env:
  - name: DB_Password
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_Password
```

### Env variables

```yaml
envFrom:
  - secretRef:
      name: MYSECRET
```

### Files in a volume

```yaml
volumes:
  - name: secret-volume
    secret:
      name: MYSECRET
```
