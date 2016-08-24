FROM nanliu/snap:0.15.0_xenial

RUN apt-get update && \
    apt-get install -y bzip2 sysstat jq less tree vim || apt-get install -y -o Acquire::http::No-Cache=True -o Acquire::BrokenProxy=true bzip2 sysstat jq less tree vim
RUN mkdir -p /tmp/plugins && \
    (cd /tmp/plugins && curl -sfLO https://github.com/intelsdi-x/snap/releases/download/v0.15.0-beta/snap-plugins-v0.15.0-beta-linux-amd64.tar.gz) && \
    (cd /tmp/plugins && tar xvf snap-plugins-v0.15.0-beta-linux-amd64.tar.gz)

RUN mkdir -p /opt/snap/examples/tasks

ADD iostat.yaml /opt/snap/examples/tasks/
