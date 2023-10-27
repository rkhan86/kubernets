# ETCD Backup & Restore process

**Step1: Set enviroment variable**

```sh
echo 'export ETCDCTL_API=3' >>~/.bashrc
```

**Step2: Install ETCD client**

```sh
apt install etcd-client
```

**Step3: Identify required values**

- Identify the below information from **/etc/kubernetes/manifests/etcd.yaml**

  - --endpoints (in etcd.yaml file --listen-client-urls value)
  - --cacert (in etcd.yaml file --trusted-ca-file value)
  - --certfile (in etcd.yaml file --cert-file value )
  - --keyfile (in etcd.yaml file --key-file value)

```sh
cat /etc/kubernetes/manifests/etcd.yaml | grep listen
cat /etc/kubernetes/manifests/etcd.yaml | grep file
```

**Step4: Run the command**

```sh
etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save <Directory_location_of_backup_file>
```

**Step5: Restore ETCD DB**

- If we check /etc/kubernetes/manifests/etcd.yaml file

```sh
cat /etc/kubernetes/manifests/etcd.yaml
```

- we will find that
  - etcd store data in a Directory
  - etcd data store inside container Directory
  - etcd data store location mounted from local host directory

```yaml
- --data-dir=/var/lib/etcd # etcd store data in a Directory
volumeMounts:
    - mountPath: /var/lib/etcd  # etcd data store inside container Directory
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd  # etcd data store location mounted from local host directory
      type: DirectoryOrCreate
    name: etcd-data
```

```sh
#restoring data in a local host directory
etcdctl snapshot restore --data-dir /var/lib/etcd-from-backup /opt/etcd_backup/etcd-1oct2023.db
# verify that data is restore
ls /var/lib/etcd-from-backup/
# change the etcd data store location mounted from local host directory
vim /etc/kubernetes/manifests/etcd.yaml

    - hostPath:
      path: /var/lib/etcd-from-backup  # etcd data store location mounted from local host directory
      type: DirectoryOrCreate
    name: etcd-data
# verify that etcd container is restarted
root@k8s-master:/opt/etcd_backup# crt ps -a | grep etcd
# output
3665753ef4688   86b6af7dd652c   5 seconds ago   Running etcd    0   38ab7f848b291       etcd-k8s-master

```

```sh
etcdctl  \
--data-dir="the_default_backup_dir" \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot restore <Directory_location_of_backup_file>
```

- If you want to use a specific data directory for the restore, you can add the location using the **--data-dir** flag as shown below.

```sh
ETCDCTL_API=3 etcdctl --data-dir /opt/etcd snapshot restore /opt/backup/etcd.db
```

vi /etc/kubernetes/manifests/etcd.yaml

```
-- data-dir= the_default_backup_dir
- mountPath= "the_default_backup_dir"
path: "the_default_backup_dir"
```
