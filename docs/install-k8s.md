## Install Kubernetes

Kubernetes is available on Google Cloud Platform Container Engine, and can be installed on multiple other platforms via kargo.

### Command Line Tools

```
export KUBERNETES_VERSION=1.3.0
curl https://storage.googleapis.com/kubernetes-release/release/v${KUBERNETES_VERSION}/bin/darwin/amd64/kubectl -o /usr/local/bin/kubectl
chmod 755 /usr/local/bin/kubectl
```

### Vagrant

Use kargo project to bootstrap kubernetes:
```
$ git clone https://github.com/intelsdi-x/kargo.git
$ cd kargo
$ git checkout idf
$ vagrant up --provider virtualbox
Bringing machine 'k8s-01' up with 'virtualbox' provider...
Bringing machine 'k8s-02' up with 'virtualbox' provider...
Bringing machine 'k8s-03' up with 'virtualbox' provider...
...
```

Verify Kubernetes is running:
```
$ vagrant ssh k8s-01 -- -L 8080:localhost:8080
$ kubectl get pods --namespace=kube-system -o wide

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

### Google Cloud Platform

Please follow the kubesnap repo for instructions on GCE:
https://github.com/intelsdi-x/kubesnap/

### Other

See Kargo project for other ways to bootstrap kubernetes:
https://docs.kubespray.io/
