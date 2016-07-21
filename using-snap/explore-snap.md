## Explore Snap

In this section we will:

* start snapd service
* load snap plugins
* create and observe task
* visualize telemetry data in Grafana

## Overview

snapd: snap service daemon
snapctl: snap commandline tool

snap plugins
* collectors
* processors
* publishers

## snapd Service

* development/debug mode (Mac)
```
$ sudo snapd -t 0 -l 1
```

* init.d (RedHat 6/Ubuntu 14.04)
```
$ sudo service snapd start
Starting snap daemon:                                      [  OK  ]
$ sudo service snapd status
snapd (pid  3362) is running...
```

* systemd (RedHat 7/Ubuntu 16.04)
```
$ sudo systemctl start snapd
$ sudo systemctl status snapd
* snapd.service - Snap daemon
   Loaded: loaded (/lib/systemd/system/snapd.service; enabled; vendor preset: e
   Active: active (running) since Tue 2016-06-07 16:19:33 PDT; 1 months 13 days
     Docs: man:snapd(8)
           man:snapctl(1)
  Process: 2869 ExecStop=/bin/kill -INT $MAINPID (code=exited, status=0/SUCCESS
 Main PID: 3149 (snapd)
    Tasks: 9 (limit: 512)
   CGroup: /system.slice/snapd.service
           `-3149 /usr/bin/snapd
```

snapd configuration files:
```
$ tree /etc/snap
/etc/snap/
|-- keyrings
`-- snapd.conf
```

examine snapd logs:
```
$ tail -f /var/log/snap/snapd.log
...
time="2016-06-07T16:19:44-07:00" level=info msg="plugin load called" _block=load _module=control
time="2016-06-07T16:19:44-07:00" level=info msg="plugin load called" _block=load-plugin _module=control-plugin-mgr path=snap-plugin-collector-cpu
time="2016-06-07T16:19:44-07:00" level=debug msg="timeout chan start" _block=waitHandling _module=control-plugin-execution
```

### Exercise

Update snapd.conf:
* set log_level=debug
* set max-running-plugins
* start snapd on a different port to verify changes.
* restart snapd service

## snapd REST API

snapctl commands corespond to snapd REST APIs:

```
$ snapctl
NAME:
   snapctl - The open telemetry framework

USAGE:
   snapctl [global options] command [command options] [arguments...]

VERSION:
   v0.15.0-beta

COMMANDS:
     agreement	Can only be used when tribe mode is enabled.
     member	Can only be used when tribe mode is enabled.
     metric
     plugin
     task
```

```
$ curl -L localhost:8181/v1/metrics
{
  "meta": {
    "code": 200,
    "message": "Metric",
    "type": "metrics_returned",
    "version": 1
  },
  "body": []
}
```

```
$ curl -L localhost:8181/v1/plugins
{
  "meta": {
    "code": 200,
    "message": "Plugin list returned",
    "type": "plugin_list_returned",
    "version": 1
  },
  "body": {}
}
```

```
$ curl -L localhost:8181/v1/tasks
{
  "meta": {
    "code": 200,
    "message": "Scheduled tasks retrieved",
    "type": "scheduled_task_list_returned",
    "version": 1
  },
  "body": {
    "ScheduledTasks": []
  }
}
```

## Load Plugin

Download plugins (IDF students skip):
```
$ curl -fL linux.plugin.dl.snap-telemetry.io -o plugin.tar.gz
$ tar xvf plugin.tar.gz
```

currently available plugins:
```
$ ls /vagrant/plugins
snap-collector-mock1                 snap-plugin-collector-nova
snap-collector-mock2                 snap-plugin-collector-openfoam
snap-plugin-collector-apache         snap-plugin-collector-osv
...
```

load plugin:
```
$ snapctl plugin load snap-plugin-collector-psutil
Plugin loaded
Name: psutil
Version: 6
Type: collector
Signed: false
Loaded Time: Thu, 21 Jul 2016 11:29:21 PDT
```

plugins expose metrics:
```
$ snapctl metric list
NAMESPACE                  VERSIONS
/intel/psutil/cpu0/guest          6
/intel/psutil/cpu0/guest_nice     6
/intel/psutil/cpu0/idle           6
/intel/psutil/cpu0/iowait         6
/intel/psutil/cpu0/irq            6
/intel/psutil/cpu0/nice           6
```

### Excercise

Use snapctl command to:
* load snap-plugin-collector-meminfo
* list meminfo metrics
* unload snap-plugin-collector-meminfo
* load snap-plugin-collector-smart

Use curl and REST API to (hint: curl ... | jq):
* list available metrics in REST
* list single metric in REST
* what does /intel/psutil/vm/inactive metrics collect? (see [metrics 2.0](http://metrics20.org/) goals)

## Writing task

We ship example task manifests.
```
$ tree /opt/snap/examples/tasks
/opt/snap/examples/tasks
|-- ceph-file.json
|-- distributed-mock-file.json
|-- mock-file.json
|-- mock-file.yaml
|-- mock_tagged-file.json
|-- psutil-file_no-processor.yaml
|-- psutil-file.yaml
|-- psutil-influx.json
`-- README.md
```

```
$ cat /opt/snap/examples/tasks/psutil-file_no-processor.yaml
---
  version: 1
  schedule:
    type: "simple"
    interval: "1s"
  max-failures: 10
  workflow:
    collect:
      metrics:
        /intel/psutil/load/load1: {}
        /intel/psutil/load/load15: {}
        /intel/psutil/load/load5: {}
      publish:
        - plugin_name: "file"
          config:
            file: "/tmp/snap_published_demo_file.log"
```

load required file publisher plugin:
```
$ snapctl plugin load snap-plugin-publisher-file
Plugin loaded
Name: file
Version: 1
Type: publisher
Signed: false
Loaded Time: Thu, 21 Jul 2016 11:31:52 PDT
```

create tasks
```
$ snapctl task create -t /opt/snap/examples/tasks/psutil-file_no-processor.yaml
Using task manifest to create task
Task created
ID: 8f3f3994-6341-49e3-bd96-10ec364e3263
Name: Task-8f3f3994-6341-49e3-bd96-10ec364e3263
State: Running

$ snapctl task list
ID                      NAME                      STATE          HIT      MISS      FAIL      CREATED          LAST FAILURE
8f3f3994-6341-49e3-bd96-10ec364e3263      Task-8f3f3994-6341-49e3-bd96-10ec364e3263      Running      16      0      0      11:41AM 7-21-2016
```

watch tasks
```
$ snapctl task watch 8f3f3994-6341-49e3-bd96-10ec364e3263
Watching Task (8f3f3994-6341-49e3-bd96-10ec364e3263):
NAMESPACE              DATA      TIMESTAMP
/intel/psutil/load/load1      0.01      2016-07-21 11:44:16.507440345 -0700 PDT
/intel/psutil/load/load15      0.05      2016-07-21 11:44:16.507451551 -0700 PDT
```

see output in file:
```
$ tail -f /tmp/snap_published_demo_file.log
[{"namespace":[{"Value":"intel","Description":"","Name":""},{"Value":"psutil","Description":"","Name":""},{"Value":"load","Description":"","Name":""},{"Value":"load1","Description":"","Name":""}],"last_advertised_time":"0001-01-01T00:00:00Z","version":0,"config":null,"data":0,"tags":{"plugin_running_on":"ubuntu1604"},"Unit_":"Load/1M","description":"","timestamp":"2016-07-21T11:49:47.705315535-07:00"},{"namespace":[{"Value":"intel","Description":"","Name":""},{"Value":"psutil","Description":"","Name":""},{"Value":"load","Description":"","Name":""},{"Value":"load15","Description":"","Name":""}],"last_advertised_time":"0001-01-01T00:00:00Z","version":0,"config":null,"data":0.05,"tags":{"plugin_running_on":"ubuntu1604"},"Unit_":"Load/15M","description":"","timestamp":"2016-07-21T11:49:47.705328457-07:00"},{"namespace":[{"Value":"intel","Description":"","Name":""},{"Value":"psutil","Description":"","Name":""},{"Value":"load","Description":"","Name":""},{"Value":"load5","Description":"","Name":""}],"last_advertised_time":"0001-01-01T00:00:00Z","version":0,"config":null,"data":0.01,"tags":{"plugin_running_on":"ubuntu1604"},"Unit_":"Load/5M","description":"","timestamp":"2016-07-21T11:49:47.705337407-07:00"}]

$ tail -1 /tmp/snap_published_demo_file.log | jq
```

### Exercise

Write a new task:
* gather all smart disk statistics. (hint use: '*' )
* write metrics to /var/log/disk_statistics.yaml
* run nightly at 3:00 AM.
* run it once to verify it works.

## Task Lifecycle

NOTE: see task lifecycle slides.

### Exercise

* delete psutil task
* disable smart task

## Grafana

### Excercise

* load snap-plugihn-publisher-influxdb
* change task to publish to influxdb
* observe telemetry data in Grafana
