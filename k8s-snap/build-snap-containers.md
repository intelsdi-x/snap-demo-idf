## Build snap containers

### kubesnap containers

Build and push snap docker images:
```
$ git clone https://github.com/intelsdi-x/kubesnap.git
$ cd kubesnap/src/heapster
$ docker build -t localhost:5000/heapster-snap .
$ docker push localhost:5000/heapster-snap
$ cd kubesnap/src/snap
$ git clone https://github.com/intelsdi-x/kubesnap-plugin-collector-docker.git
$ git clone https://github.com/intelsdi-x/kubesnap-plugin-publisher-heapster.git
$ docker build -t localhost:5000/snap .
$ docker push localhost:5000/snap
```

NOTE: replace localhost:5000 with gcr.io/${project_id} for GCE or docker hub for custom environments.
