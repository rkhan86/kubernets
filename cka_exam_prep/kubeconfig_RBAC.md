- In kubernetes **RBAC** concept you have user and group and then you have roles that define the permission of users or groups and then you bind those roles to the users or groups using a role-binding. So, role and role-binding live at a namespace level.
- To provide cluster level permission, meaning to all namespaces, you will have to use cluster role and cluster role-binding.

## User Access/Authorization

- In kubernetes does not have the concept of users. A user or group in kubernetes is just their arbitrary string. So it could be the name of the user, ID or could be a user id or group id. In traditional kubernetes, to identify a user or a group kubernetes uses a certificate and it only trust a certificate which was signed by the kubernetes CA.
- When you provision a k8s cluster you provide a CA certificate. So, this means I cann't log in to kubernetes and add a user called rzk. To do that, I have to create a client certificate and set rzk as the common name in the certificate and then I have to sign that certificate with kubernetes ca.crt and ca.key. Then I have to create a role to allow rzk some permission set and bind that role to the certificate using the same common name.

```sh
# copy the ca.crt and ca.key from control-plane's /etc/kubernetes/pki location.
#start with a private key
openssl genrsa -out rzk.key 2048
```

Now we have a key, we need a certificate signing request (CSR).</br>
We also need to specify the groups that rzk belongs to.</br>
Let's pretend rzk is part of the student team and will be developing applications for the student

```sh
openssl req -new -key rzk.key -out rzk.csr -subj "/CN=rashed khan/O=student"
```

Use the CA to generate our certificate by signing our CSR.</br>
We may set an expiry on our certificate as well

```sh
openssl x509 -req -in rzk.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out rzk.crt -days 300
```

Create a cluster, user and context entry which points to the cluster and contains the details of the CA and rzk certificate:

```sh
# set cluster
kubectl config set-cluster 131-cluster --server=https://192.168.50.131:6443 \
--certificate-authority=ca.crt \
--embed-certs=true
# set user
kubectl config set-credentials rzk --client-certificate=rzk.crt --client-key=rzk.key --embed-certs=true
# set context
kubectl config set-context 131 --cluster=131-cluster --namespace=rzk --user=rzk
# verify all set to right position
kubectl config view
kubectl config get-contexts
# to change the context
kubectl config use-context 131
# to view current context
kubectl config current-context
```

### <u>**Config File Structure**</u>

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.50.131:6443
  name: 131-cluster
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.50.151:6443
  name: kubernetes
contexts:
- context:
    cluster: 131-cluster
    namespace: rzk
    user: rzk
  name: "131"
- context:
    cluster: kubernetes
    namespace: default
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: "131"
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: rzk
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```

## RBAC Configuration

### [API-Group Reference](/cka_exam_prep/api-group-name.md)

### <u>API-Group</u>

- In Kubernetes, an apiGroup is a collection of related RESTful paths in the Kubernetes API. It’s a way to organize related kinds of resources, like Pods, Services, or Deployments. For example, the apps apiGroup contains the Deployment resource, while the core apiGroup contains the Pod resource.

```yaml
apiVersion: apps/v1
kind: Deployment
```

### <u>Resources</u>

- Resources are specific types within an apiGroup. They represent a specific object type within the Kubernetes system and are persistent entities. Examples of resources include Pods, Services, and Deployments, which exist in the core and apps apiGroups respectively.

![api-group](/logo/api-group.png "api-group")_api-group_</br>

<img src="/home/rashed/code/kubernetes/logo/api-group.png" width="900">

- Here are some common apiGroups and their associated resources:

  - core (also written as ""): This is the default apiGroup and includes resources like pods, services, endpoints, nodes, and namespaces.
  - apps: This apiGroup includes resources like deployments, replicasets, and statefulsets.
  - extensions: This apiGroup includes ingresses, daemonsets, replicasets, and more.
  - rbac.authorization.k8s.io: This apiGroup includes roles and rolebindings for RBAC.
  - batch: This apiGroup includes job, cronjob
  - autoscaling: This apiGroup includes HorizontalPodAutoscaler
  - networking.k8s.io: This apiGroup includes IngressClass, Ingress, NetworkPolicy, RuntimeClass
  - storage.k8s.io: This apiGroup includes CSIDriver, CSINode, CSIStorageCapacity, StorageClass, VolumeAttachment

- RBAC rules in Kubernetes are defined by four components: apiGroups, resources, verbs, and resourceNames. The apiGroups field specifies the group of resources the rule applies to. The resources field specifies the types of resources. The verbs field specifies the operations that can be performed on these resources, and resourceNames specifies the specific instances of the resources.
- Here’s an example of an RBAC rule:

```yaml
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

### <u> Role & RoleBinding </u>

- **Create role**

```sh
kubectl create role NAME --verb=verb --resource=resource.group/subresource [--resource-name=resourcename]
[--dry-run=server|client|none] [options]

# Examples:
  # Create a role named "pod-reader" that allows user to perform "get", "watch" and "list" on pods
  kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods

  # Create a role named "pod-reader" with ResourceName specified
  kubectl create role pod-reader --verb=get --resource=pods --resource-name=readablepod --resource-name=anotherpod

  # Create a role named "foo" with API Group specified
  kubectl create role foo --verb=get,list,watch --resource=rs.apps

  # Create a role named "foo" with SubResource specified
  kubectl create role foo --verb=get,list,watch --resource=pods,pods/status


```

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rzk-role
  namespace: rzk
rules:
  - apiGroups: [""]
    resources: ["*"] # all resources in core api group
    verbs: ["*"]
  - apiGroups: ["extensions"]
    resources: ["*"]
    verbs: ["*"]
  - apiGroups: ["apps"]
    resources: ["*"]
    verbs: ["*"]
  - apiGroups: ["autoscaling"]
    resources: ["*"]
    verbs: ["*"]
  - apiGroups: ["batch"]
    resources: ["*"]
    verbs: ["*"]
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods"]
    verbs: ["get", "list"]
```

```sh
ks apply -f rzk-role.yaml
ks get role -n rzk
ks describe -n rzk role rzk-role
```

- **Create RoleBinding**

```sh
kubectl create rolebinding NAME --clusterrole=NAME|--role=NAME [--user=username] [--group=groupname]
[--serviceaccount=namespace:serviceaccountname] [--dry-run=server|client|none] [options]

# Examples:
  # Create a role binding for user1, user2, and group1 using the admin cluster role
  kubectl create rolebinding admin --clusterrole=admin --user=user1 --user=user2 --group=group1

  # Create a role binding for serviceaccount monitoring:sa-dev using the admin role
  kubectl create rolebinding admin-binding --role=admin --serviceaccount=monitoring:sa-dev

```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rzk-rolebinding
  namespace: rzk
subjects:
  - kind: User
    name: "rashed khan" # it must need to be match with common name (CN) specified in csr file
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: rzk-role # role name that we just created
  apiGroup: rbac.authorization.k8s.io
```

```sh
ks apply -f rzk-rolebinding.yaml
ks get rolebinding -n rzk
ks describe -n rzk rolebinding rzk-rolebinding
```

### Checking API Access

```sh
kubectl auth can-i VERB [TYPE | TYPE/NAME | NONRESOURCEURL] [options]
kubectl auth can-i --list # prints all allowed actions.
kubectl auth can-i create deployments --namespace dev
kubectl auth can-i list secrets --namespace dev --as dave
kubectl auth can-i list deployment --namespace rzk --as 'rashed khan'
```

### <u> Cluster roles & cluster role bindings.</u>

- There are some resources that cann't be group or isolate within a namespace. For example node, they cannot be associated to any particular namespace. So the resources are categorized as either namespaced or cluster-scoped. The cluster-scoped resources are those where you don't specify a namespace when you create them, like nodes, persistent volumes. Cluster roles and cluster roled bindings are just like roles, except they are for cluster-scoped resources.

- You can create a cluster role for namespaced resources as well. When you do that, the user will have access to these resources across all namespaces.

- **Create a Cluster role**

```sh
# Usage:
  kubectl create clusterrole NAME --verb=verb --resource=resource.group [--resource-name=resourcename]
[--dry-run=server|client|none] [options]

Examples:
  # Create a cluster role named "pod-reader" that allows user to perform "get", "watch" and "list" on pods
  kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods

  # Create a cluster role named "pod-reader" with ResourceName specified
  kubectl create clusterrole pod-reader --verb=get --resource=pods --resource-name=readablepod
--resource-name=anotherpod

  # Create a cluster role named "foo" with API Group specified
  kubectl create clusterrole foo --verb=get,list,watch --resource=rs.apps

  # Create a cluster role named "foo" with SubResource specified
  kubectl create clusterrole foo --verb=get,list,watch --resource=pods,pods/status

  # Create a cluster role name "foo" with NonResourceURL specified
  kubectl create clusterrole "foo" --verb=get --non-resource-url=/logs/*

  # Create a cluster role name "monitoring" with AggregationRule specified
  kubectl create clusterrole monitoring --aggregation-rule="rbac.example.com/aggregate-to-monitoring=true"
```

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rzk-cluster-role
rules:
  - apiGroups: [""]
    resources: ["nodes", "persistentvolume", "namespaces"]
    verbs: ["list", "get", "create", "delete"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingressclasses"]
    verbs: ["create", "delete", "get", "list", "watch"]
  - apiGroups: ["metrics.k8s.io"]
    resources: ["nodes"]
    verbs: ["get", "list"]
```

```sh
ks apply -f rzk-cluster-role.yaml
ks get clusterrole
ks describe clusterrole rzk-cluster-role
```

- **Create a cluster RoleBinding**

```sh
# Usage:
  kubectl create clusterrolebinding NAME --clusterrole=NAME [--user=username] [--group=groupname]
[--serviceaccount=namespace:serviceaccountname] [--dry-run=server|client|none] [options]
Examples:
  # Create a cluster role binding for user1, user2, and group1 using the cluster-admin cluster role
  kubectl create clusterrolebinding cluster-admin --clusterrole=cluster-admin --user=user1 --user=user2 --group=group1
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: rzk-cluster-rolebinding
subjects:
  - kind: User
    name: "rashed khan" # it must need to be match with common name (CN) specified in csr file
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: rzk-cluster-role # role name that we just created
  apiGroup: rbac.authorization.k8s.io
```

```sh
ks apply -f rzk-cluster-rolebinding.yaml
ks get clusterrolebinding
ks describe clusterrolebinding  rzk-cluster-rolebinding
```

### <u> Service Account</u>

- So there are two types of accounts in Kubernetes. A user account and a service account. As you might already know, the user account is used by humans. And service accounts are used by machines. A user account could be for an administrator accessing the cluster to perform administrative tasks, a developer accessing the cluster to deploy applications etc. A service account, could be an account used by an application to interact with the kubernetes cluster. For example a monitoring application like Prometheus uses a service account to poll the kubernetes API for performance metrics. An automated build tool like Jenkins uses service accounts to deploy applications on the kubernetes cluster.

**How to use service accounts**

- Create a ServiceAccount object using a Kubernetes client like kubectl or a manifest that defines the object.
- Grant permissions to the ServiceAccount object using an authorization mechanism such as RBAC.
- Assign the ServiceAccount object to Pods during Pod creation.

**Create serviceaccount**

```sh
kubectl create serviceaccount dashboard-sa
kubectl get serviceaccount
kubectl describe serviceaccount dashboard-sa
```

### Task:

Create a new ClusterRole named deployment-clusterrole, which only allows to create the following resource types

- Deployment
- Stateful Set
- Deamon Set

create a new service account cicd-token in thr existing namespace app-team1 </br>
bind the cluster role with new service account, limited to the namespace app-team1

```sh
kubectl create clusterrole deployment-clusterrole --verb=create --resource=Deployment,StatefulSet,Deamon Set
kubectl create serviceaccount cicd-token --namespace app-team1
kubectl create clusterrolebinding deployment-clusterrole --clusterrole=deployment-clusterrole
--serviceaccount=app-team1:cicd-token
```

### <u>For More Detailes</u>

- [Service Accounts](https://kubernetes.io/docs/concepts/security/service-accounts/)
- [Managing Service Accounts](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)
- [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
