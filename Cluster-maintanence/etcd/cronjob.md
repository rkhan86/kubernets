# Kubernetes: How to effectively backup etcd

**Introduction**

- In order to restore cluster successfully from failure, we create scheduled backup job. Kubernetes natively support cronjobs, so we are going to use that feature for our workflow.

- Workflow is divided into three stages:
    - Build backup tool
    - Deploy cronjob
    - Delete old backup

**Stage:1 Build Docker Image**

- Create Dockerfile with a following content (see below). Notice “ARG” parameter, this helps us to add version control our builtin tool.

```Dockerfile
FROM alpine:latest

ARG ETCD_VERSION=v3.4.27

ENV ETCDCTL_ENDPOINTS "https://127.0.0.1:2379"
ENV ETCDCTL_CACERT "/etc/kubernetes/pki/etcd/ca.crt"
ENV ETCDCTL_KEY "/etc/kubernetes/pki/etcd/healthcheck-client.key"
ENV ETCDCTL_CERT "/etc/kubernetes/pki/etcd/healthcheck-client.crt"

RUN apk add --update --no-cache bash ca-certificates tzdata openssl

RUN wget https://github.com/etcd-io/etcd/releases/download/${ETCD_VERSION}/etcd-${ETCD_VERSION}-linux-amd64.tar.gz \
 && tar xzf etcd-${ETCD_VERSION}-linux-amd64.tar.gz \
 && mv etcd-${ETCD_VERSION}-linux-amd64/etcdctl /usr/local/bin/etcdctl \
 && rm -rf etcd-${ETCD_VERSION}-linux-amd64*

ENTRYPOINT ["/bin/bash"]
```

- Export “ETCD_VERSION” variable, which will be used in next step. Changing tool version can be simply done by changing variable value.

```sh
export ETCD_VERSION=v3.4.13
```

- Below command will build our backup image.

```sh
docker build --build-arg=$ETCD_VERSION -t etcd-backup:$ETCD_VERSION .
```

- Verify that built image exists!

```sh
docker images | grep -i "etcd-backup"
```

**Stage:2 Backup Etcd Datastore Using Kubernetes Cronjob**

- Now that we have built backup container, lets create scheduled backup cronjob. For demonstration purpose we use “default” namespace.

- Create file named etcd-backup-cronjob.yaml and paste below content. Notice last line, where we mount host timezone into our container.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: etcd-backup
  namespace: kube-system
spec:
  schedule: "*/2 * * * *"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 5
  concurrencyPolicy: Allow
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: etcd-backup
            image: rzkhan/etcd-backup:v3.4.27
            imagePullPolicy: IfNotPresent
            resources: {}
            env:
            - name: ETCDCTL_API
              value: "3"
            - name: ETCDCTL_ENDPOINTS
              value: "https://127.0.0.1:2379"
            - name: ETCDCTL_CACERT
              value: "/etc/kubernetes/pki/etcd/ca.crt"
            - name: ETCDCTL_CERT
              value: "/etc/kubernetes/pki/etcd/healthcheck-client.crt"
            - name: ETCDCTL_KEY
              value: "/etc/kubernetes/pki/etcd/healthcheck-client.key"
            command: ["/bin/bash","-c"]
            args: ["etcdctl snapshot save /opt/etcd-backup/etcd-snapshot-$(date +%Y-%m-%dT%H:%M).db"]
            volumeMounts:
            - mountPath: /etc/kubernetes/pki/etcd
              name: etcd-certs
              readOnly: true
            - mountPath: /opt/etcd-backup
              name: etcd-backup
            - mountPath: /etc/localtime
              name: local-timezone
          - name: backup-purge
            image: busybox:1.36.1
            imagePullPolicy: IfNotPresent
            resources: {}
            args: 
            - -c
            - find /opt/etcd-backup -type f -mtime +30 -name '*.db' -exec rm -fr {} \;
            # - find /opt/etcd-backup -type f -mtime +30 -name '*.db' -exec rm -- '{}' \;
            command: 
            - /bin/bash
            volumeMounts:
            - mountPath: /opt/etcd-backup
              name: etcd-backup
          restartPolicy: OnFailure
          dnsPolicy: ClusterFirst
          hostNetwork: true
          nodeSelector:
            node-role.kubernetes.io/control-plane: ""
          schedulerName: default-scheduler
          securityContext: {}
          terminationGracePeriodSeconds: 30
          tolerations:
          - effect: NoSchedule
            operator: Exists
          volumes:
          - name: etcd-certs
            hostPath:
              path: /etc/kubernetes/pki/etcd
              type: Directory
          - name: etcd-backup
            hostPath:
              path: /opt/etcd-backup
              type: DirectoryOrCreate
          - name: local-timezone
            hostPath:
              path: /usr/share/zoneinfo/Asia/Dhaka
```

- Deploy cronjob to cluster.

```sh
kubectl apply -f etcd-backup-cronjob.yaml
```

- Verify that cronjob exits in cluster with below command.

```sh
kubectl get cronjobs -n kube-system
```