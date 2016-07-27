## Snap with Grafana

In this section we will:

* launch heapster&grafana application
* observe snap metrics in grafana

### Launch grafana service

Create granfana service
```
$ kubectl create -f ./heapster
```

Find and access grafana dashboard:
```
$ kubectl cluster-info | grep grafana
```

### Exercise

* Observe kubernetes metrics per node
* Observe grafana container running metrics


### Live monitoring

* Upgrade snap plugin
