
# Source-to-Image Workshop Content

This document contains the code/configuration snippets corresponding to the workshop slides.
## Option 1 - Dockerfile
### Dockerfile - Step 1
```shell
#SSH to your instance using the provided key.
[user@localhost] ssh –i <keyfile> ec2-user@studentXX.s2i.rhtps.io

#Echo the $GOPATH variable.
[ec2-user@ip-10-0-0-XX ~]$ echo $GOPATH
/home/ec2-user/golang

#Change directory to $GOPATH
[ec2-user@ip-10-0-0-XX ~]$ cd $GOPATH
```
### Dockerfile - Step 2
```bash
#Obtain the source code for "gochat".  This downloads all dependencies as well.
[ec2-user@ip-10-0-0-XX ~]$ go get github.com/rhtps/gochat

[ec2-user@ip-10-0-0-XX ~]$ cd $GOPATH/src/github.com/rhtps/gochat

[ec2-user@ip-10-0-0-XX gochat]$ go build

#Start "gochat" binding 8080 to the host's interface.  The callbackHost is relevant if you use an omniauth provider like github.
[ec2-user@ip-10-0-0-XX gochat]$ ./gochat -host=0.0.0.0:8080 -callBackHost=studentXX.s2i.rhtps.io:8080 -templatePath=$GOPATH/src/github.com/rhtps/gochat/templates/ -githubProviderKey=<provided> -githubProviderSecretKey=<provided>
#OR
[ec2-user@ip-10-0-0-XX gochat]$ ./gochat -host=0.0.0.0:8080
```
### Dockerfile - Step 3
```bash
[ec2-user@ip-10-0-0-XX ~]$ cd ~

[ec2-user@ip-10-0-0-XX ~]$ git clone https://github.com/rhtps/gochat-docker.git

[ec2-user@ip-10-0-0-XX gochat-docker]$ cd gochat-docker

[ec2-user@ip-10-0-0-XX gochat-docker]$ vi Dockerfile
#Take note of the directives.  Quit when you are finished.

[ec2-user@ip-10-0-0-XX gochat-docker]$ sudo systemctl start docker

[ec2-user@ip-10-0-0-XX gochat-docker]$ docker build -t chat .
```
### Dockerfile - Step 4
```bash
[ec2-user@ip-10-0-0-XX gochat-docker]$ docker run -d -p 8080:8080 --name gochat chat -host=0.0.0.0:8080 -callBackHost=student01.s2i.rhtps.io:8080 -templatePath=/opt/golang/github.com/rhtps/gochat/templates -avatarPath=/opt/golang/github.com/rhtps/gochat/avatars

[ec2-user@ip-10-0-0-XX gochat-docker]$ docker stop gochat

#Note the stopped container
[ec2-user@ip-10-0-0-XX gochat-docker]$ docker ps -a

[ec2-user@ip-10-0-0-XX gochat-docker]$ docker rm gochat
```
---
## Option 2 - Source-to-Image (S2I)
### S2I - Step 1
```bash
[ec2-user@ip-10-0-0-XX ~]$ go get github.com/openshift/source-to-image

[ec2-user@ip-10-0-0-XX ~]$ cd $GOPATH/src/github.com/openshift/source-to-image

[ec2-user@ip-10-0-0-XX source-to-image]$ export PATH=$PATH:$GOPATH/src/github.com/openshift/source-to-image/_output/local/bin/linux/amd64/

[ec2-user@ip-10-0-0-XX source-to-image]$ hack/build-go.sh
```
### S2I - Step 2
```bash
[ec2-user@ip-10-0-0-XX ~]$ cd ~

[ec2-user@ip-10-0-0-XX ~]$ mkdir golang-s2i

[ec2-user@ip-10-0-0-XX ~]$ cd golang-s2i

[ec2-user@ip-10-0-0-XX golang-s2i]$ s2i create golang-s2i .

[ec2-user@ip-10-0-0-XX golang-s2i]$ tree -a
.
├── Dockerfile
├── Makefile
├── .s2i
│   └── bin
│       ├── assemble
│       ├── run
│       ├── save-artifacts
│       └── usage
└── test
    ├── run
    └── test-app
```
### S2I - Step 3
```bash
[ec2-user@ip-10-0-0-XX golang-s2i]$ vi Dockerfile
```
And then paste the following Content
```bash
# golang-s2i
FROM rhel7

MAINTAINER Kenneth D. Evensen <kevensen@redhat.com>

ENV BUILDER_VERSION 1.0
ENV GOPATH /opt/go
ENV GOBIN /opt/go/bin
ENV PATH $PATH:$GOBIN

LABEL io.k8s.description="Platform for building go based programs" \
      io.k8s.display-name="gobuilder 0.0.1" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="gobuilder,0.0.1" \
      io.openshift.s2i.scripts-url="image://usr/local/s2i" \
      io.openshift.s2i.destination="/opt/go/destination"

RUN yum install -y --disablerepo='*' --enablerepo='rhel-7-server-rpms' --enablerepo='rhel-7-server-optional-rpms' golang tar git-bzr && yum clean all; rm -rf /var/cache/yum

COPY ./.s2i/bin/ /usr/local/s2i
RUN useradd 1001
RUN mkdir -p /opt/go/destination/{src,artifacts}; chown -R 1001:1001 /opt/

USER 1001

EXPOSE 8080
```
### S2I - Step 4
```bash
[ec2-user@ip-10-0-0-XX golang-s2i]$ vi .s2i/bin/assemble
#!/bin/bash -e #
# S2I assemble script for the 'golang-s2i' image.
cd $GOPATH/destination/src
go get -v .
cd $GOBIN
mv src goexec
```
### S2I - Step 5
```bash
[ec2-user@ip-10-0-0-XX golang-s2i]$ vi .s2i/bin/run
#!/bin/bash –e
exec $GOBIN/goexec $ARG

```
