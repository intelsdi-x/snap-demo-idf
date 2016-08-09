## Snap in Kubernetes

In this section we will:

* launch snap daemonset service
* interact with snap tribe

### Snap configuration

review snap configuration file:
```
$ cat ~/kargo/configs/snap/*.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: snap-config
  namespace: kube-system
data:
  tribe.seed: '172.17.8.101'
  tribe.nodes: '3'
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: snap
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
spec:
  template:
    metadata:
      name: snap
      labels:
        daemon: snapd
...
```

deploy snap application:
```
$ kubectl create -f ~/kargo/configs/snap/
configmap "snap-config" created
daemonset "snap" created
```

### Exercise:

* deploy snap into kubernetes
* What snap plugins and tasks are running?
* Use snap watch to observe a running task

### Snap Tribe

```
$ snapctl member list
$ snapctl agreement list
```

### Exercise

* Create a new snap cpu task and verify it's deployed to all members.
* Create a new agreement and add tribe member.
