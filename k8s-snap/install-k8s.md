## Install Kubernetes

In this section we will:

* install kubectl tools locally
* start kuberenets VMs
* verify kubernetes services are running.

NOTE: commands executed in the vagrant VMs are prefaced with `k8s-01 $` instead of just `$`

### Command Line Tools

Install kubectl command line tool:
```
mkdir "${HOME}/bin"
export KUBERNETES_VERSION=1.3.0

if [[ $(uname) == "Darwin" ]]
then export OS="darwin"
else export OS="linux"
fi

curl "https://storage.googleapis.com/kubernetes-release/release/v${KUBERNETES_VERSION}/bin/${OS}/amd64/kubectl" -o "${HOME}/bin/kubectl"

chmod 755 "${HOME}/bin/kubectl"
```

Install kubectl autocompletion:
```
kubectl completion zsh > "${HOME}/.kubectl.zsh"
echo "source ${HOME}/.kubectl.zsh" >> "${HOME}/.zshrc"
source ${HOME}/.zshrc
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

NOTE: during initial provisioning, some tasks such as etcd may error, but should  startup correctly unless the verification steps fail. 

Verify Kubernetes is running:
```
$ vagrant ssh k8s-01 -- -L 8080:localhost:8080

k8s-01 $ kubectl cluster-info
Kubernetes master is running at http://localhost:8080
dnsmasq is running at http://localhost:8080/api/v1/proxy/namespaces/kube-system/services/dnsmasq
kubedns is running at http://localhost:8080/api/v1/proxy/namespaces/kube-system/services/kubedns

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

k8s-01 $ kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-2               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```

Verify etcd is running and flannel network available (one per VM):
```
k8s-01 $ etcdctl ls /cluster.local/network/subnets
/cluster.local/network/subnets/10.233.71.0-24
/cluster.local/network/subnets/10.233.109.0-24
/cluster.local/network/subnets/10.233.85.0-24
```

Run vagrant provision again if cluster-info does not show kubedns running or no flannel network is listed:
```
$ vagrant provision
```

The ssh port forwarding will allow kubectl command to work directly the local system without the need to ssh into k8s-01.

```
$ kubectl cluster-info
Kubernetes master is running at http://localhost:8080
dnsmasq is running at http://localhost:8080/api/v1/proxy/namespaces/kube-system/services/dnsmasq
kubedns is running at http://localhost:8080/api/v1/proxy/namespaces/kube-system/services/kubedns
```

## Reference

Kubernetes is available on Google Cloud Platform Container Engine, and can be installed on multiple other platforms via kargo:

https://docs.kubespray.io/
