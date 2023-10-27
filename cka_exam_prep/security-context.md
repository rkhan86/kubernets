# Security Context in Kubernetes

- When you run a Docker container, you have the option to define a set of security standards,such as the ID of the user used to run the container,the Linux capabilities that can be added or removed from the container, et cetera.These can be configured in Kubernetes as well.

- As you know already, in Kubernetes,containers are encapsulated in Pods.You may choose to configure the security settings at a container level or at a Pod level.If you configure it at a Pod level,the settings will carry over to all the containers within the Pod.If you configure it at both the Pod and the container,the settings on the container will override the settings on the Pod.

## Set the security context for a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
    - name: sec-ctx-vol
      emptyDir: {}
  containers:
    - name: sec-ctx-demo
      image: busybox:1.28
      command: ["sh", "-c", "sleep 1h"]
      volumeMounts:
        - name: sec-ctx-vol
          mountPath: /data/demo
      securityContext:
        allowPrivilegeEscalation: false
```

In the configuration file, the runAsUser field specifies that for any Containers in the Pod, all processes run with user ID 1000. The runAsGroup field specifies the primary group ID of 3000 for all processes within any containers of the Pod. If this field is omitted, the primary group ID of the containers will be root(0). Any files created will also be owned by user 1000 and group 3000 when runAsGroup is specified. Since fsGroup field is specified, all processes of the container are also part of the supplementary group ID 2000. The owner for volume /data/demo and any files created in that volume will be Group ID 2000.

## Set the security context for a Container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-2
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: sec-ctx-demo-2
      image: gcr.io/google-samples/node-hello:1.0
      securityContext:
        runAsUser: 2000
        allowPrivilegeEscalation: false
```

**AllowPrivilegeEscalation:** Controls whether a process can gain more privileges than its parent process. This bool directly controls whether the no_new_privs flag gets set on the container process. AllowPrivilegeEscalation is true always when the container is: 1) run as Privileged OR 2) has CAP_SYS_ADMIN.

**fsGroupChangePolicy** - fsGroupChangePolicy defines behavior for changing ownership and permission of the volume before being exposed inside a Pod. This field only applies to volume types that support fsGroup controlled ownership and permissions. This field has two possible values:

- OnRootMismatch: Only change permissions and ownership if the permission and the ownership of root directory does not match with expected permissions of the volume. This could help shorten the time it takes to change ownership and permission of a volume.

- Always: Always change permission and ownership of the volume when volume is mounted.

- For example:

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
  fsGroupChangePolicy: "OnRootMismatch"
```

## Set capabilities for a Container

- With Linux capabilities, you can grant certain privileges to a process without granting all the privileges of the root user. To add or remove Linux capabilities for a Container, include the capabilities field in the securityContext section of the Container manifest.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-4
spec:
  containers:
    - name: sec-ctx-4
      image: gcr.io/google-samples/node-hello:1.0
      securityContext:
        capabilities:
          add: ["NET_ADMIN", "SYS_TIME"]
```

**For Details:**
["NET_ADMIN", "SYS_TIME"](https://github.com/torvalds/linux/blob/master/include/uapi/linux/capability.h)
