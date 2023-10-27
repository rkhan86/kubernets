# Renewing your Kubernetes Certificate

```sh
# Take backup of the exsisting cluster sertificate and configuration
mkdir /home/user/cert_backup
cp -p /etc/kubernetes/*.conf  /home/user/cert_backup
cp -pr /etc/kubernetes/pki  /home/user/cert_backup

----------------------------------------------
# Identify the expiry data
kubeadm certs check-expiration
kubectl cluster-info
echo | openssl s_client -showcerts -connect <Kubernetes_control_plane_IP>:6443 -servername api 2>/dev/null | openssl x509 -noout --enddate
kubectl -n kube-system get cm kubeadm-config -o yaml

----------------------------------------------
# See all option s for kubeadm certs renew command
kubeadm certs renew --help
# Renew all certificate
kubeadm certs renew all
# Restart controller, apiserver, scheduler, etcd
crictl pods
crictl stopp <pod_ID> && crictl rmp <pod_ID>

```

```sh
#To view the the information of a cirtificate
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep -i not
```
