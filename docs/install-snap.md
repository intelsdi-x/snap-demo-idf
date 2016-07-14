## Install Snap

* MacOS
* RedHat/Centos
* Ubuntu

## MacOS

```
curl -O https://github.com/intelsdi-x/snap/releases/download/v0.15.0-beta/snap-telemetry-0.15.0.pkg
sudo installer -allowUntrusted -verboseR -pkg ./snap-telemetry-0.15.0.pkg -target /
```

## Redhat

```
curl -s https://packagecloud.io/install/repositories/intelsdi-x/snap/script.rpm.sh | sudo bash
sudo yum install -y snap-telemetry
```

## Ubuntu

```
curl -s https://packagecloud.io/install/repositories/intelsdi-x/snap/script.deb.sh | sudo bash
sudo apt-get install -y snap-telemetry
```
