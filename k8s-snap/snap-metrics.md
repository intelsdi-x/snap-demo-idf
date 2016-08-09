# Snap with Grafana

In this section we will:

* launch heapster & grafana application
* observe snap metrics in grafana

## Launch grafana service

Create granfana service
```
$ kubectl create -f ~/kargo/configs/heapster
```

Find and access grafana dashboard:
```
$ kubectl cluster-info | grep grafana
```

### Exercise

* Observe kubernetes metrics per node
* Observe grafana container running metrics

## Autoscale Pods

kubernetes Horizontal Pod Autoscaling (HPA) uses heapster to monitor pod activity and automatically creates/destroys pods on demand.


create php application:
```
$ kubectl run php-apache --image=gcr.io/google_containers/hpa-example --requests=cpu=200m --expose --port=80
service "php-apache" created
deployment "php-apache" created
```

set autoscale desired range:
```
$ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=3
deployment "php-apache" autoscaled
```

review current status:
```
$ kubectl get hpa
NAME         REFERENCE                     TARGET    CURRENT   MINPODS   MAXPODS   AGE
php-apache   Deployment/php-apache/scale   50%       0%        1         10        18s
```

increase load:
```
$ kubectl run -i --tty load-generator --image=busybox /bin/sh

Hit enter for command prompt

$ while true; do wget -q -O- http://php-apache.default.svc.cluster.local; doneA
```

observe hpa respond to load increase:
```
$ kubectl get hpa
NAME         REFERENCE                     TARGET    CURRENT   MINPODS   MAXPODS   AGE
php-apache   Deployment/php-apache/scale   50%       305%      1         10        3m
```

### Exercise

* remove load (terminate load-generator)
* observe hpa status and workload in grafana (NOTE: select the correct kubernetes namespace)
