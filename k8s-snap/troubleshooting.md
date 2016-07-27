### Local registry issues

The local registry service is only required for IDF lab:

verify the registry service is running and snap images are available:
```
$ curl localhost:5000/v2/_catalog
{"repositories":[]}
```

recreate the registry service if there's an issue:
```
$ kubectl delete -f ./registry
$ kubectl create -f ./registry
daemonset "kube-registry-proxy" created
persistentvolume "kube-system-kube-registry-pv" created
persistentvolumeclaim "kube-registry-pvc" created
service "kube-registry" created
replicationcontroller "kube-registry-v0" created
```

wait until the kube-registry pods are all in running status:
```
$ kubectl get pod --namespace=kube-system | grep kube-registry
kube-registry-proxy-6vulv        1/1       Running             0          2m        10.233.76.3    k8s-01
kube-registry-proxy-qxbf2        1/1       Running             0          2m        10.233.88.4    k8s-02
kube-registry-proxy-yxscq        1/1       Running             0          2m        10.233.99.3    k8s-03
kube-registry-v0-1dl0l           0/1       ContainerCreating   0          2m        <none>         k8s-03
```

for details on pod progress we can run kubectl describe
```
$ kubectl describe pods --namespace=kube-system kube-registry-v0-1dl0l
Name:        kube-registry-v0-1dl0l
Namespace:   kube-system
Node:        k8s-03/172.17.8.103
Start Time:  Thu, 21 Jul 2016 14:24:58 -0700
Labels:      k8s-app=kube-registry
    kubernetes.io/cluster-service=true
    version=v0
Status:      Pending
IP:
Controllers: ReplicationController/kube-registry-v0
Containers:
  registry:
    Container ID:
    Image:    registry:2
    Image ID:
    Port:     5000/TCP
    Limits:
      cpu:      100m
      memory:   100Mi
    Requests:
      cpu:      100m
      memory:   100Mi
    State:      Waiting
      Reason:     ContainerCreating
    Ready:      False
    Restart Count:  0
    Environment Variables:
      REGISTRY_HTTP_ADDR:                         :5000
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY:  /var/lib/registry
Conditions:
  Type          Status
  Initialized   True
  Ready         False
  PodScheduled  True
Volumes:
  image-store:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  kube-registry-pvc
    ReadOnly:   false
  default-token-q4xll:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-q4xll
QoS Tier:  Guaranteed
Events:
  FirstSeen  LastSeen  Count  From      SubobjectPath    Type    Reason           Message
  ---------  --------  -----  ----      -------------    --------  ------    -------
  2m    2m    1  {default-scheduler }        Normal    Scheduled        Successfully assigned kube-registry-v0-1dl0l to k8s-03
  34s    34s    1  {kubelet k8s-03}  spec.containers{registry}Normal    Pulling           pulling image "registry:2"
```
