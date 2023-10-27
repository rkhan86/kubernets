# [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

## Access Modes

A PersistentVolume can be mounted on a host in any way supported by the resource provider. As shown in the table below, providers will have different capabilities and each PV's access modes are set to the specific modes supported by that particular volume. For example, NFS can support multiple read/write clients, but a specific NFS PV might be exported on the server as read-only. Each PV gets its own set of access modes describing that specific PV's capabilities.

The access modes are:</br>

**ReadWriteOnce :**
the volume can be mounted as read-write by a single node. ReadWriteOnce access mode still can allow multiple pods to access the volume when the pods are running on the same node.</br>
**ReadOnlyMany :**
the volume can be mounted as read-only by many nodes.</br>
**ReadWriteMany :**
the volume can be mounted as read-write by many nodes.</br>
**ReadWriteOncePod :**</br>
**FEATURE STATE: Kubernetes v1.27 [beta]**</br>
the volume can be mounted as read-write by a single Pod. Use ReadWriteOncePod access mode if you want to ensure that only one pod across the whole cluster can read that PVC or write to it. This is only supported for CSI volumes and Kubernetes version 1.22+.</br>

The blog article Introducing Single Pod Access Mode for PersistentVolumes covers this in more detail.</br>

In the CLI, the access modes are abbreviated to:</br>

    RWO - ReadWriteOnce
    ROX - ReadOnlyMany
    RWX - ReadWriteMany
    RWOP - ReadWriteOncePod

## Reclaim Policy

Current reclaim policies are:</br>

    Retain -- manual reclamation
    Recycle -- basic scrub (rm -rf /thevolume/*)
    Delete -- associated storage asset such as AWS EBS or GCE PD volume is deleted

Currently, only NFS and HostPath support recycling. AWS EBS and GCE PD volumes support deletion.</br>

## Volume Mode

FEATURE STATE: Kubernetes v1.18 [stable]</br>

Kubernetes supports two volumeModes of PersistentVolumes: Filesystem and Block.</br>

A volume with **volumeMode: Filesystem** is mounted into Pods into a directory. If the volume is backed by a block device and the device is empty, Kubernetes creates a filesystem on the device before mounting it for the first time.</br>

You can set the value of volumeMode to Block to use a volume as a raw block device. Such volume is presented into a Pod as a block device, without any filesystem on it. This mode is useful to provide a Pod the fastest possible way to access a volume, without any filesystem layer between the Pod and the volume. On the other hand, the application running in the Pod must know how to handle a raw block device. See Raw Block Volume Support for an example on how to use a volume with volumeMode: Block in a Pod.</br>

**<span style="color: red;">Note:</span>** </br>
volumeMode is an optional API parameter. Filesystem is the default mode used when volumeMode parameter is omitted. </br>

## Volume Binding Mode

The volumeBindingMode field controls when volume binding and dynamic provisioning should occur. When unset, "Immediate" mode is used by default.</br>

The **Immediate** mode indicates that volume binding and dynamic provisioning occurs once the PersistentVolumeClaim is created.</br>

The **WaitForFirstConsumer** mode which will delay the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created </br>

## [StorageClassName](https://kubernetes.io/docs/concepts/storage/storage-classes/)

A PV can have a class, which is specified by setting the storageClassName attribute to the name of a StorageClass. A PV of a particular class can only be bound to PVCs requesting that class. A PV with no storageClassName has no class and can only be bound to PVCs that request no particular class.</br>

## Create a storage class with Local Provisioner

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true" # This make Default storageclass for cluster
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```

Local volumes do not currently support dynamic provisioning, however a StorageClass should still be created to delay volume binding until Pod scheduling. This is specified by the WaitForFirstConsumer volume binding mode.

Delaying volume binding allows the scheduler to consider all of a Pod's scheduling constraints when choosing an appropriate PersistentVolume for a PersistentVolumeClaim.

- To make a storage class use as default for cluste, below command:

```sh
# Show all storageclass
kubectl get storageclass
# Mark a StorageClass as default:
kubectl patch storageclass <storageclass-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
# Mark the default StorageClass as non-default:
kubectl patch storageclass <storageclass-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
# for detailes
ks describe storageclasses.storage.k8s.io <storageclass-name>
```

## Persistent storage using hostPath

- Kubernetes supports hostPath for development and testing on a single-node cluster. A hostPath PersistentVolume uses a file or directory on the Node to emulate network-attached storage.

### [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

- Here is the configuration file for the hostPath PersistentVolume:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvol1
  labels:
    type: local
spec:
  storageClassName: local-storage
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data1"
```

- View information about the PersistentVolume:

```sh
kubectl get pv pvol1
```

- Here is the configuration file for the PersistentVolumeClaim:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvclm1
  namespace: cka
spec:
  storageClassName: local-storage
  selector:
    matchLabels:
      type: local
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

- Look at the PersistentVolumeClaim:

```sh
kubectl get pvc pvclm1
```

- The next step is to create a Pod that uses your PersistentVolumeClaim as a volume. Here is the configuration file for the Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: webhost
  name: webpod1
  namespace: cka
spec:
  volumes:
    - name: pvol1
      persistentVolumeClaim:
        claimName: pvclm1
  containers:
    - image: nginx
      name: webcon1
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pvol1
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
