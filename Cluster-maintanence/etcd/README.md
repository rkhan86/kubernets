# ETCD Backup process

**Step1: Set enviroment variable**

```sh
echo 'export ETCDCTL_API=3' >>~/.bashrc
```

**Step2: Install ETCD client**

```sh
apt install etcd-client
```
**Step3: Identify required values**
- Identify the below information from /etc/kubernetes/manifests/etcd.yaml

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
snapshort save <Directory_location_of_backup_file>
```

**Step5: Restore ETCD DB**

```sh
etcdctl  \
--data-dir="the_default_backup_dir" \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshort save <Directory_location_of_backup_file>
```

vi /etc/kubernetes/manifests/etcd.yaml

```
-- data-dir= the_default_backup_dir
- mountPath= "the_default_backup_dir"
path: "the_default_backup_dir"
```