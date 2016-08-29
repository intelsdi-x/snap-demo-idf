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
k8s-01 $ kubectl create -f /vagrant/configs/snap/
configmap "snap-config" created
daemonset "snap" created
```

### Exercise:

* deploy snap into kubernetes
* What snap plugins and tasks are running? (install snap package in the k8s-01 VM via the command below):
    ```
k8s-01 $ curl -s https://packagecloud.io/install/repositories/intelsdi-x/snap/script.deb.sh | sudo bash
Detected operating system as Ubuntu/xenial.
Checking for curl...
Detected curl...
Running apt-get update... done.
Installing apt-transport-https... done.
Installing /etc/apt/sources.list.d/intelsdi-x_snap.list...done.
Importing packagecloud gpg key... done.
Running apt-get update... done.

The repository is setup! You can now install packages.

k8s-01 $ sudo apt-get install snap-telemetry
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  snap-telemetry
  0 upgraded, 1 newly installed, 0 to remove and 96 not upgraded.
  Need to get 7,725 kB of archives.
  After this operation, 26.6 MB of additional disk space will be used.
  Get:1 https://packagecloud.io/intelsdi-x/snap/ubuntu xenial/main amd64 snap-telemetry amd64 0.15.0-1xenial [7,725 kB]
  Fetched 7,725 kB in 11s (655 kB/s)
  Selecting previously unselected package snap-telemetry.
  (Reading database ... 94475 files and directories currently installed.)
  Preparing to unpack .../snap-telemetry_0.15.0-1xenial_amd64.deb ...
  Unpacking snap-telemetry (0.15.0-1xenial) ...
  Processing triggers for man-db (2.7.5-1) ...
  Setting up snap-telemetry (0.15.0-1xenial) ...
    ```
* Use snap watch to observe the running task

### Snap Tribe

```
k8s-01 $ snapctl member list
Name
k8s-01
k8s-02
k8s-03

k8s-01 $ snapctl agreement list
Name           Number of Members   plugins            tasks
all-nodes      3                   3                  1
```

### Exercise

* Use snapctl to add a plugin and verify it's deployed to all nodes.
