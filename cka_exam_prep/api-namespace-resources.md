```sh
ks api-resources -o wide --namespaced=true
```

```
NAME                        SHORTNAMES                                      APIVERSION                     NAMESPACED   KIND                       VERBS                                                        CATEGORIES
bindings                                                                    v1                             true         Binding                    create
configmaps                  cm                                              v1                             true         ConfigMap                  create,delete,deletecollection,get,list,patch,update,watch
endpoints                   ep                                              v1                             true         Endpoints                  create,delete,deletecollection,get,list,patch,update,watch
events                      ev                                              v1                             true         Event                      create,delete,deletecollection,get,list,patch,update,watch
limitranges                 limits                                          v1                             true         LimitRange                 create,delete,deletecollection,get,list,patch,update,watch
persistentvolumeclaims      pvc                                             v1                             true         PersistentVolumeClaim      create,delete,deletecollection,get,list,patch,update,watch
pods                        po                                              v1                             true         Pod                        create,delete,deletecollection,get,list,patch,update,watch   all
podtemplates                                                                v1                             true         PodTemplate                create,delete,deletecollection,get,list,patch,update,watch
replicationcontrollers      rc                                              v1                             true         ReplicationController      create,delete,deletecollection,get,list,patch,update,watch   all
resourcequotas              quota                                           v1                             true         ResourceQuota              create,delete,deletecollection,get,list,patch,update,watch
secrets                                                                     v1                             true         Secret                     create,delete,deletecollection,get,list,patch,update,watch
serviceaccounts             sa                                              v1                             true         ServiceAccount             create,delete,deletecollection,get,list,patch,update,watch
services                    svc                                             v1                             true         Service                    create,delete,deletecollection,get,list,patch,update,watch   all
controllerrevisions                                                         apps/v1                        true         ControllerRevision         create,delete,deletecollection,get,list,patch,update,watch
daemonsets                  ds                                              apps/v1                        true         DaemonSet                  create,delete,deletecollection,get,list,patch,update,watch   all
deployments                 deploy                                          apps/v1                        true         Deployment                 create,delete,deletecollection,get,list,patch,update,watch   all
replicasets                 rs                                              apps/v1                        true         ReplicaSet                 create,delete,deletecollection,get,list,patch,update,watch   all
statefulsets                sts                                             apps/v1                        true         StatefulSet                create,delete,deletecollection,get,list,patch,update,watch   all
localsubjectaccessreviews                                                   authorization.k8s.io/v1        true         LocalSubjectAccessReview   create
horizontalpodautoscalers    hpa                                             autoscaling/v2                 true         HorizontalPodAutoscaler    create,delete,deletecollection,get,list,patch,update,watch   all
cronjobs                    cj                                              batch/v1                       true         CronJob                    create,delete,deletecollection,get,list,patch,update,watch   all
jobs                                                                        batch/v1                       true         Job                        create,delete,deletecollection,get,list,patch,update,watch   all
leases                                                                      coordination.k8s.io/v1         true         Lease                      create,delete,deletecollection,get,list,patch,update,watch
networkpolicies                                                             crd.projectcalico.org/v1       true         NetworkPolicy              delete,deletecollection,get,list,patch,create,update,watch
networksets                                                                 crd.projectcalico.org/v1       true         NetworkSet                 delete,deletecollection,get,list,patch,create,update,watch
endpointslices                                                              discovery.k8s.io/v1            true         EndpointSlice              create,delete,deletecollection,get,list,patch,update,watch
events                      ev                                              events.k8s.io/v1               true         Event                      create,delete,deletecollection,get,list,patch,update,watch
pods                                                                        metrics.k8s.io/v1beta1         true         PodMetrics                 get,list
ingresses                   ing                                             networking.k8s.io/v1           true         Ingress                    create,delete,deletecollection,get,list,patch,update,watch
networkpolicies             netpol                                          networking.k8s.io/v1           true         NetworkPolicy              create,delete,deletecollection,get,list,patch,update,watch
poddisruptionbudgets        pdb                                             policy/v1                      true         PodDisruptionBudget        create,delete,deletecollection,get,list,patch,update,watch
networkpolicies             cnp,caliconetworkpolicy,caliconetworkpolicies   projectcalico.org/v3           true         NetworkPolicy              create,delete,deletecollection,get,list,patch,update,watch
networksets                 netsets                                         projectcalico.org/v3           true         NetworkSet                 create,delete,deletecollection,get,list,patch,update,watch
rolebindings                                                                rbac.authorization.k8s.io/v1   true         RoleBinding                create,delete,deletecollection,get,list,patch,update,watch
roles                                                                       rbac.authorization.k8s.io/v1   true         Role                       create,delete,deletecollection,get,list,patch,update,watch
csistoragecapacities                                                        storage.k8s.io/v1              true         CSIStorageCapacity         create,delete,deletecollection,get,list,patch,update,watch
```
