## Install Snap

For simplicy, the lab will use snap within a docker container. This section will ensure the containers are installed and running for the lab:

### Build and run Snap containers

Clone Snap IDF demo repository:
```
$ git clone git@github.com:intelsdi-x/snap-demo-idf.git
```

Build Snap docker images:
```
$ cd snap-demo-idf
$ docker-compose build
Building snap
Step 1 : FROM nanliu/snap:0.15.0_xenial
 ---> 356bcaa4fcbb
 Step 2 : RUN mkdir -p /tmp/plugins &&     (cd /tmp/plugins && curl -sfLO https://github.com/intelsdi-x/snap/releases/download/v0.15.0-beta/snap-plugins-v0.15.0-beta-linux-amd64.tar.gz) &&     (cd /tmp/plugins && tar xvf snap-plugins-v0.15.0-beta-linux-amd64.tar.gz)
  ---> Running in f7c498bffdbc
  snap-v0.15.0-beta/
  snap-v0.15.0-beta/plugin/
  snap-v0.15.0-beta/plugin/snap-plugin-collector-apache
  snap-v0.15.0-beta/plugin/snap-plugin-collector-ceph
  snap-v0.15.0-beta/plugin/snap-plugin-collector-cinder
  snap-v0.15.0-beta/plugin/snap-plugin-collector-cpu
  snap-v0.15.0-beta/plugin/snap-plugin-collector-dbi
  snap-v0.15.0-beta/plugin/snap-plugin-collector-df
  snap-v0.15.0-beta/plugin/snap-plugin-collector-disk
  snap-v0.15.0-beta/plugin/snap-plugin-collector-docker
  snap-v0.15.0-beta/plugin/snap-plugin-collector-elasticsearch
  snap-v0.15.0-beta/plugin/snap-plugin-collector-etcd
  snap-v0.15.0-beta/plugin/snap-plugin-collector-ethtool
  snap-v0.15.0-beta/plugin/snap-plugin-collector-facter
  snap-v0.15.0-beta/plugin/snap-plugin-collector-glance
  snap-v0.15.0-beta/plugin/snap-plugin-collector-haproxy
  snap-v0.15.0-beta/plugin/snap-plugin-collector-influxdb
  snap-v0.15.0-beta/plugin/snap-plugin-collector-interface
  snap-v0.15.0-beta/plugin/snap-plugin-collector-iostat
  snap-v0.15.0-beta/plugin/snap-plugin-collector-keystone
  snap-v0.15.0-beta/plugin/snap-plugin-collector-libvirt
  snap-v0.15.0-beta/plugin/snap-plugin-collector-load
  snap-v0.15.0-beta/plugin/snap-plugin-collector-meminfo
  snap-v0.15.0-beta/plugin/snap-plugin-collector-mesos
  snap-v0.15.0-beta/plugin/snap-plugin-collector-mock1
  snap-v0.15.0-beta/plugin/snap-plugin-collector-mock2
  snap-v0.15.0-beta/plugin/snap-plugin-collector-mysql
  snap-v0.15.0-beta/plugin/snap-plugin-collector-neutron
  snap-v0.15.0-beta/plugin/snap-plugin-collector-nfsclient
  snap-v0.15.0-beta/plugin/snap-plugin-collector-node-manager
  snap-v0.15.0-beta/plugin/snap-plugin-collector-nova
  snap-v0.15.0-beta/plugin/snap-plugin-collector-openfoam
  snap-v0.15.0-beta/plugin/snap-plugin-collector-osv
  snap-v0.15.0-beta/plugin/snap-plugin-collector-pcm
  snap-v0.15.0-beta/plugin/snap-plugin-collector-perfevents
  snap-v0.15.0-beta/plugin/snap-plugin-collector-processes
  snap-v0.15.0-beta/plugin/snap-plugin-collector-psutil
  snap-v0.15.0-beta/plugin/snap-plugin-collector-rabbitmq
  snap-v0.15.0-beta/plugin/snap-plugin-collector-scaleio
  snap-v0.15.0-beta/plugin/snap-plugin-collector-smart
  snap-v0.15.0-beta/plugin/snap-plugin-collector-swap
  snap-v0.15.0-beta/plugin/snap-plugin-collector-users
  snap-v0.15.0-beta/plugin/snap-plugin-processor-movingaverage
  snap-v0.15.0-beta/plugin/snap-plugin-processor-passthru
  snap-v0.15.0-beta/plugin/snap-plugin-processor-tag
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-file
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-graphite
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-hana
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-influxdb
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-kafka
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-kairosdb
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-mock-file
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-mysql
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-opentsdb
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-postgresql
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-rabbitmq
  snap-v0.15.0-beta/plugin/snap-plugin-publisher-riemann
   ---> dcf749cb20d2
   Removing intermediate container f7c498bffdbc
   Successfully built dcf749cb20d2
   grafana uses an image, skipping
```

Start docker containers:
```
$ docker-compose up -d
Creating snap
Creating grafana
```

Verify Snap & Grafana containers are running:
```
$  docker-compose ps
Name                Command               State           Ports
-------------------------------------------------------------------------
grafana   /run.sh                          Up      0.0.0.0:3000->3000/tcp
snap      /bin/sh -c /opt/snap/bin/s ...   Up      0.0.0.0:8181->8181/tcp
```

Snap service logs are available via:
```
$ docker-compose logs snap
Attaching to snap
snap       | time="2016-08-24T17:10:03Z" level=info msg="setting log level to: debug"
snap       | time="2016-08-24T17:10:03Z" level=info msg="Starting snapd (version: v0.15.0-beta)"
snap       | time="2016-08-24T17:10:03Z" level=info msg="setting GOMAXPROCS to: 1 core(s)"
snap       | time="2016-08-24T17:10:03Z" level=debug msg="pevent controller created" _block=new _module=control
...
```

### Cleanup

When you are finished with the lab, simply run:
```
$ docker-compose stop
$ docker-compose rm
```
