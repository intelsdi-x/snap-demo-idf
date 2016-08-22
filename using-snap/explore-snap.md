# Explore Snap

In this section we will:

* configure and start snapd service
* manage snap plugins
* determine available telemetry metrics
* create and observe task
* visualize telemetry data in Grafana

## Overview

snap commands:
* snapd: snap service daemon
* snapctl: snap command line tool

## `snapd` service

Start snap daemon using the appropriate service command:

* init.d service:
```
$ service snapd start
$ service snapd status
```

* systemd service:
```
$ sudo systemctl daemon-reload
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

Update snapd.conf (edit as root: `sudo vim /etc/snap/snapd.conf`):
* set `log_level: 1` (debug log)
* set `max_running_plugins: 5`
* open a new terminal and monitor log file: `tail -f /var/log/snap/snapd.log`
* restart snapd service and verify changes in log file output:

    NOTE: if snapd service fails to start, troubleshoot in development debug mode (ctrl+c to exit):
    ```
$ sudo snapd -l 1
    ```

## `snapctl` command

snapctl contains several sub-commands:

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

download and extract snap plugins:
```
$ mkdir -p ${HOME}/idflab/
$ cd ${HOME}/idflab/
$ curl -fL https://github.com/intelsdi-x/snap/releases/download/v0.15.0-beta/snap-plugins-v0.15.0-beta-linux-amd64.tar.gz -o snap-plugins.tar.gz
$ tar xvf snap-plugins.tar.gz
```

list available plugins (see [Plugin Catalog](http://snap-telemetry.io/plugins.html) for current list and plugins in development).
```
$ cd snap-v0.15.0-beta/plugin/
$ ls
snap-plugin-collector-apache         snap-plugin-collector-nova
snap-plugin-collector-ceph           snap-plugin-collector-openfoam
snap-plugin-collector-cinder         snap-plugin-collector-osv 
...
```

load snap psutil plugin:
```
$ snapctl plugin load snap-plugin-collector-psutil
Plugin loaded
Name: psutil
Version: 6
Type: collector
Signed: false
Loaded Time: Thu, 21 Jul 2016 11:29:21 PDT
```

list metrics exposed by plugins:
```
$ snapctl metric list | less
NAMESPACE                  VERSIONS
/intel/psutil/cpu0/guest          6
/intel/psutil/cpu0/guest_nice     6
/intel/psutil/cpu0/idle           6
/intel/psutil/cpu0/iowait         6
/intel/psutil/cpu0/irq            6
/intel/psutil/cpu0/nice           6
```

### Exercise

Extract snap plugins, and use snapctl command to:
* load `snap-plugin-collector-meminfo` plugin
* list meminfo metrics (hint: `... | grep meminfo`)
* unload `snap-plugin-collector-meminfo` (hint: use `snapctl plugin list` to get necessary info)
* load `snap-plugin-collector-smart` plugin
* load `snap-plugin-publisher-file` plugin

## snap REST API

snapctl is retrieving data from snapd REST API:
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

### Exercise

Use curl and REST API to:
* list all available metrics in REST (hint: `curl ... | jq`)
* list a single metric in REST
* what does /intel/psutil/vm/inactive metrics collect? (for more info see [metrics 2.0](http://metrics20.org/))

## Running telemetry tasks

snap package includes several example tasks:
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

review an example task:
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

create task from config file:
```
$ snapctl task create -t /opt/snap/examples/tasks/psutil-file_no-processor.yaml
Using task manifest to create task
Task created
ID: 8f3f3994-6341-49e3-bd96-10ec364e3263
Name: Task-8f3f3994-6341-49e3-bd96-10ec364e3263
State: Running
```

list all tasks:
```
$ snapctl task list
ID                                        NAME                                           STATE        HIT     MISS   FAIL   CREATED              LAST FAILURE
8f3f3994-6341-49e3-bd96-10ec364e3263      Task-8f3f3994-6341-49e3-bd96-10ec364e3263      Running      16      0      0      11:41AM 7-21-2016
```

watch running tasks:
```
$ snapctl task watch 8f3f3994-6341-49e3-bd96-10ec364e3263
Watching Task (8f3f3994-6341-49e3-bd96-10ec364e3263):
NAMESPACE                     DATA      TIMESTAMP
/intel/psutil/load/load1      0.01      2016-07-21 11:44:16.507440345 -0700 PDT
/intel/psutil/load/load15     0.05      2016-07-21 11:44:16.507451551 -0700 PDT
```

see output in file:
```
$ tail -f /tmp/snap_published_demo_file.log
[{"namespace":[{"Value":"intel","Description":"","Name":""},{"Value":"psutil","Description":"","Name":""},{"Value":"load","Description":"","Name":""},{"Value":"load1","Description":"","Name":""}],"last_advertised_time":"0001-01-01T00:00:00Z","version":0,"config":null,"data":0,"tags":{"plugin_running_on":"ubuntu1604"},"Unit_":"Load/1M","description":"","timestamp":"2016-07-21T11:49:47.705315535-07:00"},{"namespace":[{"Value":"intel","Description":"","Name":""},{"Value":"psutil","Description":"","Name":""},{"Value":"load","Description":"","Name":""},{"Value":"load15","Description":"","Name":""}],"last_advertised_time":"0001-01-01T00:00:00Z","version":0,"config":null,"data":0.05,"tags":{"plugin_running_on":"ubuntu1604"},"Unit_":"Load/15M","description":"","timestamp":"2016-07-21T11:49:47.705328457-07:00"},{"namespace":[{"Value":"intel","Description":"","Name":""},{"Value":"psutil","Description":"","Name":""},{"Value":"load","Description":"","Name":""},{"Value":"load5","Description":"","Name":""}],"last_advertised_time":"0001-01-01T00:00:00Z","version":0,"config":null,"data":0.01,"tags":{"plugin_running_on":"ubuntu1604"},"Unit_":"Load/5M","description":"","timestamp":"2016-07-21T11:49:47.705337407-07:00"}]

$ tail -1 /tmp/snap_published_demo_file.log | jq
```

stop task:
```
$ snapctl task stop 8f3f3994-6341-49e3-bd96-10ec364e3263
Task stopped:
ID: 8f3f3994-6341-49e3-bd96-10ec364e3263
```

### Exercise

* ensure the following plugins are load
    * `snap-plugin-collector-psutil`
    * `snap-plugin-publisher-file`
* run the example task `/opt/snap/examples/tasks/psutil-file_no-processor.yaml`
* verify the task is running successfully (`tail -1 ... | jq`)
* stop the psutil task `snapctl task stop ...`

## Writing tasks

snap task processes telemetry data correspond with the three types of plugins availble:
* collectors
* processors
* publishers

### task scheduling

snap schedule supports:
* simple (run forever on given interval)
* window (repeat interval between start/stop time)
* cron (use cron syntax for interval)

cron schedule example:
```
---
  version: 1
  schedule: {
      type: cron
      interval: "0 30 * * * *"
  },
```

### task metrics

snap collect metrics can be a concrete namespace, a wildcard `*`, or a tuple `(a|b)`.

As an example the mock plugin with the metrics:
* /intel/mock/foo/ex1
* /intel/mock/foo/ex2
* /intel/mock/bar/ex1
* /intel/mock/bar/ex2


| metrics in task manifest | collected metrics |
| ------------------------ | ----------------- |
| /intel/mock/\* | /intel/mock/foo/ex1 </br> /intel/mock/foo/ex2 </br> /intel/mock/bar/ex1 </br> /intel/mock/bar/ex2 |
| /intel/mock/foo/(ex1/ex2) | /intel/mock/foo/ex1 </br> /intel/mock/foo/ex2|
| /intel/mock/\*/ex1) | /intel/mock/foo/ex1 </br> /intel/mock/bar/ex1 |

### Exercise

* gather all smart disk statistics. (use: `snap-plugin-collector-smart`)
* write metrics to /var/log/disk_statistics.log
* schedule the task to run every minute
* wait and verify the task ran successfully

## Grafana

Grafana provides real time visualization of telemetry data gathered by snap.

resume the Grafana docker container:
```
$ docker pull grafana/grafana:3.1.0
$ docker run 
```

* login to Grafana at [http://localhost:3000](http://localhost:3000) (user: admin password: admin)
* navigate to the [app config page](http://localhost:3000/plugins/raintank-snap-app/edit) and enable snap app:
    ![enable snap](../images/enable_snap.gif)

* click on the Grafana logo and [select data sources](http://localhost:3000/datasources)
    ![add datasource](../images/datasource.gif)
    create a [new data source](http://localhost:3000/datasources/new) use the following settings:

    ```
Name: "snap"
Default: true (checkbox)
Type: Snap DS
URL: http://localhost:8181
Access: proxy
    ```
* create a [new dashboard](http://localhost:3000/dashboard/new)
    ![create dashboard](../images/dashboard.gif)
    * add a graph panel: (insert screenshot)
    * select 'snap' as Panel datasource: (insert screenshot)
    * create the following task:
    ```
Task Name: memory active
Interval: 200ms
Metrics: /intel/procfs/meminfo/active
    ```
* click on watch and observe the metrics stream in

### Exercise

* login to grafana and create the snap datasource
* create a new dashboard and monitor cpu metrics
