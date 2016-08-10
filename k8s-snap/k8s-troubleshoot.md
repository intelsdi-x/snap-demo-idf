## Troubleshooting

### kubernetes API does not respond:
```
k8s-01 $ kubectl cluster-info
The connection to the server localhost:8080 was refused - did you specify the right host or port?
k8s-01 $ exit
```
solution: restore vagrant vm snapshot:
```
$ vagrant halt -f && vagrant snapshot pop
==> k8s-01: Restoring the snapshot 'push_1470351011_6318'...
==> k8s-01: Deleting the snapshot 'push_1470351011_6318'...
==> k8s-01: Snapshot deleted!
...
```

### kubernetes pod hanging in completed state
```
k8s-01 $ kubectl get pods --all-namespaces
                 NAME                 READY     STATUS              RESTARTS   AGE       NODE
kube-system      kubedns-r0dns        0/4       Completed           0          2m        10.233.72.4
```
solution: force a redeploy (only if it is stuck in completed status)
```
k8s-01 $ kubectl delete pod kubedns-r0dns --namespace=kube-system
```

### kubernetes pod image pull error
```
k8s-01 $ kubectl get pods --all-namespaces
                 NAME                             READY     STATUS             RESTARTS   AGE       NODE
kube-system      kube-registry-proxy-r0dns        0/1       ErrImagePull       0          2m        10.233.72.4
kube-system      kube-registry-proxy-rk44t        0/1       ImagePullBackOff   0          2m        10.233.71.3
...
```
debug: kubectl describe pod shows [docker pull failure](https://github.com/docker/docker/issues/23184):
```
k8s-01 $ kubectl describe pod kube-registry-proxy-m5rpu --namespace=kube-system
Name:		kube-registry-proxy-m5rpu
Namespace:	kube-system
Node:		k8s-01/172.17.8.101
Start Time:	Fri, 05 Aug 2016 00:22:16 +0000
Labels:		daemon=kube-registry-proxy
Status:		Pending
IP:		10.233.117.3
Controllers:	DaemonSet/kube-registry-proxy
Containers:
  kube-registry-proxy:
    Container ID:
    Image:		gcr.io/google_containers/kube-registry-proxy:0.3
    Image ID:
    Port:		5000/TCP
    Limits:
      cpu:	100m
      memory:	50Mi
    Requests:
      cpu:		100m
      memory:		50Mi
    State:		Waiting
      Reason:		ImagePullBackOff
    Ready:		False
    Restart Count:	0
    Environment Variables:
      REGISTRY_HOST:	kube-registry.kube-system.svc.cluster.local
      REGISTRY_PORT:	5000
      FORWARD_PORT:	5000
Conditions:
  Type		Status
  Initialized 	True
  Ready 	False
  PodScheduled 	True
Volumes:
  default-token-nbf4n:
    Type:	Secret (a volume populated by a Secret)
    SecretName:	default-token-nbf4n
QoS Tier:	Guaranteed
Events:
  FirstSeen	LastSeen	Count	From			SubobjectPath				Type		Reason		Message
  ---------	--------	-----	----			-------------				--------	------		-------
  6m		6m		1	{kubelet k8s-01}	spec.containers{kube-registry-proxy}	Warning		Failed		Failed to pull image "gcr.io/google_containers/kube-registry-proxy:0.3": image pull failed for gcr.io/google_containers/kube-registry-proxy:0.3, this may be because there are no credentials on this request.  details: (Error pulling image (0.3) from gcr.io/google_containers/kube-registry-proxy, failed to register layer: rename /var/lib/docker/image/aufs/layerdb/tmp/layer-978436924 /var/lib/docker/image/aufs/layerdb/sha25
```
solution: cleanup bad docker aufs layers:
```
$ vagrant ssh k8s-01 -c 'sudo find /var/lib/docker/image/aufs/layerdb/sha256/ -size 0 -exec dirname {} \; | uniq | xargs sudo rm -rf'
$ vagrant ssh k8s-02 -c 'sudo find /var/lib/docker/image/aufs/layerdb/sha256/ -size 0 -exec dirname {} \; | uniq | xargs sudo rm -rf'
$ vagrant ssh k8s-03 -c 'sudo find /var/lib/docker/image/aufs/layerdb/sha256/ -size 0 -exec dirname {} \; | uniq | xargs sudo rm -rf'
```

### missing docker registry and docker register proxy pod
```
k8s-01 $ kubectl get pods --namespace=kube-system -o wide

NAME                             READY     STATUS    RESTARTS   AGE       NODE
dnsmasq-k1qkq                    1/1       Running   0          1h        k8s-02
dnsmasq-kez7o                    1/1       Running   0          1h        k8s-01
dnsmasq-x48kg                    1/1       Running   0          1h        k8s-03
flannel-k8s-01                   2/2       Running   4          1h        k8s-01
flannel-k8s-02                   2/2       Running   2          1h        k8s-02
flannel-k8s-03                   2/2       Running   3          1h        k8s-03
kube-controller-manager-k8s-01   1/1       Running   0          1h        k8s-01
kube-controller-manager-k8s-02   1/1       Running   1          1h        k8s-02
kube-proxy-k8s-01                1/1       Running   2          1h        k8s-01
kube-proxy-k8s-02                1/1       Running   2          1h        k8s-02
kube-proxy-k8s-03                1/1       Running   2          1h        k8s-03
kube-scheduler-k8s-01            1/1       Running   0          1h        k8s-01
kube-scheduler-k8s-02            1/1       Running   0          1h        k8s-02
kubedns-cvsbm                    4/4       Running   0          1h        k8s-03
```
solution: deploy the local docker registry
```
k8s-01 $ kubectl create -f /vagrant/configs/registry
```
