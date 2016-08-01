## Install Snap

list package content:
```
$ dpkg-deb -c ${home}/idflab/snap-telemetry_0.15.0-1xenial_amd64.deb
drwxr-xr-x 0/0               0 2016-07-15 16:21 ./
drwxr-xr-x 0/0               0 2016-07-15 16:21 ./opt/
drwxr-xr-x 0/0               0 2016-07-15 16:21 ./opt/snap/
drwxr-xr-x 0/0               0 2016-07-15 16:21 ./opt/snap/bin/
-rwxr-xr-x 0/0        16977919 2016-07-15 16:21 ./opt/snap/bin/snapd
-rwxr-xr-x 0/0         9554554 2016-07-15 16:21 ./opt/snap/bin/snapctl
...

install snap:

```
$ sudo dpkg -i ${home}/idflab/snap-telemetry_0.15.0-1xenial_amd64.deb
(Reading database ... 59710 files and directories currently installed.)
Preparing to unpack snap-telemetry_0.15.0-1xenial_amd64.deb ...
Unpacking snap-telemetry (0.15.0-1xenial) over (0.15.0-1xenial) ...
Setting up snap-telemetry (0.15.0-1xenial) ...
Processing triggers for man-db (2.7.5-1) ...
```

snap man pages:
```
$ man snapd
$ man snapctl
```
