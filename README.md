
# Source-to-Image Workshop Content

This document contains the code/configuration snippets corresponding to the workshop slides.

<!-- TOC depthFrom:2 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Pre Docker](#pre-docker)
	- [Pre Docker - Step 1](#pre-docker-step-1)
	- [Pre Docker - Step 2](#pre-docker-step-2)
	- [Pre Docker - Step 3](#pre-docker-step-3)
	- [Pre Docker - Step 5](#pre-docker-step-5)
	- [Pre Docker - Step 6](#pre-docker-step-6)
- [Option 1 - Dockerfile](#option-1-dockerfile)
	- [Dockerfile - Step 1](#dockerfile-step-1)
	- [Dockerfile - Step 2](#dockerfile-step-2)
	- [Dockerfile - Step 3](#dockerfile-step-3)
	- [Dockerfile - Step 4](#dockerfile-step-4)
- [Option 2 - Source-to-Image (S2I)](#option-2-source-to-image-s2i)
	- [S2I - Step 1](#s2i-step-1)
	- [S2I - Step 2](#s2i-step-2)
	- [S2I - Step 3](#s2i-step-3)
	- [S2I - Step 4](#s2i-step-4)
	- [S2I - Step 5](#s2i-step-5)
	- [S2I - Step 6](#s2i-step-6)
	- [S2I - Step 7](#s2i-step-7)
	- [S2I - Step 8](#s2i-step-8)
	- [S2I - Step 9](#s2i-step-9)
	- [S2I - Step 10](#s2i-step-10)
- [Preparing for OpenShift](#preparing-for-openshift)
	- [Template](#template)
	- [ImageStreams](#imagestreams)
	- [BuildConfiguration](#buildconfiguration)
	- [DeploymentConfiguration](#deploymentconfiguration)
	- [Services](#services)
	- [Route](#route)
	- [Parameters](#parameters)
	- [Final](#final)

<!-- /TOC -->
## Pre Docker
### Pre Docker - Step 1
Set the environment variables
```shell
[student@localhost ~]$ cd ~

[student@localhost ~]$ cat <<EOF >> ~/.bashrc
export GOROOT=$HOME/go
export GOPATH=$HOME/work
export GOBIN=$HOME/work/bin
export PATH=$PATH:$HOME/work/src/github.com/openshift/source-to-image/_output/local/bin/linux/amd64/:$HOME/go/bin:$HOME/work/bin:$HOME/bzr-2.7.0
EOF

[student@localhost ~]$ source ~/.bashrc
```
### Pre Docker - Step 2
```shell
# Obtain golang tooling
[student@localhost ~]$ curl -k -o go1.6.2.linux-amd64.tar.gz https://storage.googleapis.com/golang/go1.6.2.linux-amd64.tar.gz

[student@localhost ~]$ tar -xvf go1.6.2.linux-amd64.tar.gz
```
### Pre Docker - Step 3
```shell
# Obtain Canonical Bazaar
[student@localhost ~]$ curl -k -L -o bzr.tar.gz https://launchpad.net/bzr/2.7/2.7.0/+download/bzr-2.7.0.tar.gz

[student@localhost ~]$ tar -xvf bzr.tar.gz

```
### Pre Docker - Step 5
```shell
#Download and build the gochat application
[student@localhost ~]$ cd ~

[student@localhost ~]$ go get -insecure github.com/rhtps/gochat

[student@localhost ~]$ go install github.com/rhtps/gochat
```
### Pre Docker - Step 6
```shell
[student@localhost ~]$ gochat -host=localhost:8080 -callBackHost=http://localhost:8080 -templatePath=$GOPATH/src/github.com/rhtps/gochat/templates/ -avatarPath=$GOPATH/src/github.com/rhtps/gochat/avatars -htpasswdPath=$GOPATH/src/github.com/rhtps/gochat/htpasswd
```
## Option 1 - Dockerfile
### Dockerfile - Step 1
```shell
[student@localhost ~]$ cd ~

[student@localhost ~]$ git clone https://github.com/rhtps/gochat-docker.git

[student@localhost ~]$ less gochat-docker/Dockerfile
```
### Dockerfile - Step 2
```shell
# Build the image
[student@localhost ~]$ cd ~/gochat-docker

[student@localhost gochat-docker]$ docker build -t gochat-docker .
```
### Dockerfile - Step 3
```shell
# Start the gochat container
[student@localhost ~]$ docker run -d -p 8080:8080 --name gochat gochat-docker -host=0.0.0.0:8080 -callBackHost=http://0.0.0.0:8080 -templatePath=/opt/gopath/src/github.com/rhtps/gochat/templates -avatarPath=/opt/gopath/src/github.com/rhtps/gochat/avatars -htpasswdPath=/opt/gopath/src/github.com/rhtps/gochat/htpasswd

[student@localhost ~]$ docker ps
```
### Dockerfile - Step 4
```shell
[student@localhost ~]$ docker stop gochat

[student@localhost ~]$ sudo docker rm gochat
```
---
## Option 2 - Source-to-Image (S2I)
### S2I - Step 1
```shell
[student@localhost ~]$ go get github.com/openshift/source-to-image

[student@localhost ~]$ cd $GOPATH/src/github.com/openshift/source-to-image

[student@localhost source-to-image]$ export PATH=$PATH:$GOPATH/src/github.com/openshift/source-to-image/_output/local/bin/linux/amd64/

[student@localhost source-to-image]$ hack/build-go.sh
```
### S2I - Step 2
```shell
[student@localhost ~]$ cd ~

[student@localhost ~]$ s2i create golang-s2i golang-s2i

[student@localhost ~]$ tree -a golang-s2i
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
```shell
[student@localhost ~]$ cat /dev/null > ~/golang-s2i/Dockerfile

[student@localhost ~]$ vi ~/golang-s2i/Dockerfile
```
And then paste the following Content
```docker
# golang-s2i
FROM centos:7

MAINTAINER Kenneth D. Evensen <kevensen@redhat.com>

ARG GO_VERSION=1.6.2
ENV BUILDER_VERSION 1.0
ENV HOME /opt/app-root
ENV GOPATH $HOME/gopath
ENV GOBIN $GOPATH/bin
ENV GOROOT $HOME/go
ENV PATH $PATH:$GOROOT/bin:$GOBIN

LABEL io.k8s.description="Platform for building go based programs" \
      io.k8s.display-name="gobuilder 0.0.1" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="gobuilder,0.0.1" \
      io.openshift.s2i.scripts-url="image:///usr/local/s2i" \
      io.openshift.s2i.destination="/opt/app-root/destination"

RUN yum clean all && \
    yum install -y tar git-bzr && \
		yum clean all && \
		rm -rf /var/cache/yum/*

COPY ./.s2i/bin/ /usr/local/s2i
RUN useradd -u 1001 -r -g 0 -d ${HOME} -s /sbin/nologin -c "Default Application User" default && \
    mkdir -p /opt/app-root/destination/{src,artifacts} && \
    curl -o $HOME/go${GO_VERSION}.linux-amd64.tar.gz https://storage.googleapis.com/golang/go${GO_VERSION}.linux-amd64.tar.gz && \
    tar -C $HOME -xvf $HOME/go${GO_VERSION}.linux-amd64.tar.gz && \
    rm -f $HOME/go${GO_VERSION}.linux-amd64.tar.gz && \
    chown -R 1001:0 $HOME && chmod -R og+rwx ${HOME}

WORKDIR ${HOME}
USER 1001
EXPOSE 8080
```
### S2I - Step 4
```shell
[student@localhost ~]$ cat /dev/null > ~/golang-s2i/.s2i/bin/assemble

[student@localhost ~]$ vi ~/golang-s2i/.s2i/bin/assemble
```
And ensure the file matches the following
```shell
#!/bin/bash
# S2I assemble script for the 'golang-s2i' image.
cd $HOME/destination/src
go get -insecure .
cd $GOBIN
mv src goexec
```
### S2I - Step 5
```shell
[student@localhost ~]$ cat /dev/null > ~/golang-s2i/.s2i/bin/run

[student@localhost ~]$ vi ~/golang-s2i/.s2i/bin/run
```
And ensure the file matches the following
```shell
#!/bin/bash
exec $GOBIN/goexec $ARGS
```
### S2I - Step 6
```shell
[student@localhost ~]$ cat /dev/null > ~/golang-s2i/.s2i/bin/save-artifacts

[student@localhost ~]$ vi ~/golang-s2i/.s2i/bin/save-artifacts
```
And ensure the file matches the following
```shell
#!/bin/bash

cd $GOPATH
tar cf - pkg bin src
```
### S2I - Step 7
```shell
[student@localhost ~]$ cat /dev/null > ~/golang-s2i/.s2i/bin/usage

[student@localhost ~]$ vi ~/golang-s2i/.s2i/bin/usage
```
### S2I - Step 8
```shell
[student@localhost ~]$ cd ~/golang-s2i/

[student@localhost golang-s2i]$ docker build -t golang-s2i .
```
### S2I - Step 9
```shell
# s2i build <source location> <builder image> [<tag>] [flags]

[student@localhost golang-s2i]$ s2i build https://github.com/rhtps/gochat.git golang-s2i gochat
```
### S2I - Step 10
```shell
[student@localhost golang-s2i]$ docker run -e "ARGS=-callBackHost=http://localhost:8080 -templatePath=/opt/app-root/gopath/src/github.com/rhtps/gochat/templates -avatarPath=/opt/app-root/gopath/src/github.com/rhtps/gochat/avatars -htpasswdPath=/opt/app-root/gopath/src/github.com/rhtps/gochat/htpasswd" -p 8080:8080 -d --name chat gochat

[student@localhost golang-s2i]$ docker ps
```
When you are done
```shell
[student@localhost golang-s2i]$ docker stop chat

[student@localhost golang-s2i]$ docker rm chat
```
---
## Preparing for OpenShift
### Template
```yaml
apiVersion: v1
kind: Template
labels:
  template: golang
metadata:
  annotations:
    description: A basic builder for Golang applications
    iconClass: icon-go-gopher
    tags: golang
  creationTimestamp: null
  name: golang
objects:
  ...
```
### ImageStreams
```yaml
objects:
- apiVersion: v1
  kind: ImageStream
  metadata:
    annotations:
      description: Keeps track of changes in the application image
    name: golang-builder
- apiVersion: v1
  kind: ImageStream
  metadata:
    annotations:
      description: Keeps track of changes in the application image
    name: ${APPLICATION_NAME}
```
### BuildConfiguration
```yaml
objects:
…
- apiVersion: v1
  kind: BuildConfig
  metadata:
    annotations:
      description: Defines how to build the application
    name: golang-builder
  spec:
    output:
      to:
        kind: ImageStreamTag
        name: golang-builder:latest
    source:
      contextDir: ${BUILDER_CONTEXT_DIR}
      git:
        ref: ${BUILDER_SOURCE_REPOSITORY_REF}
        uri: ${BUILDER_SOURCE_REPOSITORY_URL}
      type: Git
    strategy:
      dockerStrategy:
      type: Docker
    triggers:
    - type: ConfigChange
    - github:
        secret: ${BUILDER_GITHUB_WEBHOOK_SECRET}
      type: GitHub
- apiVersion: v1
  kind: BuildConfig
  metadata:
    annotations:
      description: Defines how to build the application
    name: ${APPLICATION_NAME}
  spec:
    output:
      to:
        kind: ImageStreamTag
        name: ${APPLICATION_NAME}:latest
    source:
      contextDir: ${APP_CONTEXT_DIR}
      git:
        ref: ${APP_SOURCE_REPOSITORY_REF}
        uri: ${APP_SOURCE_REPOSITORY_URL}
      type: Git
    strategy:
      sourceStrategy:
        from:
          kind: ImageStreamTag
          name: golang-builder:latest
      type: Source
    triggers:
    - type: ImageChange
    - type: ConfigChange
    - github:
        secret: ${APP_GITHUB_WEBHOOK_SECRET}
      type: GitHub
```
### DeploymentConfiguration
```yaml
- apiVersion: v1
  kind: DeploymentConfig
  metadata:
    annotations:
      description: Defines how to deploy the application server
    name: ${APPLICATION_NAME}
  spec:
    replicas: 1
    selector:
      name: ${APPLICATION_NAME}
    strategy:
      type: Rolling
    template:
      metadata:
        labels:
          name: ${APPLICATION_NAME}
        name: ${APPLICATION_NAME}
      spec:
        containers:
        - env:
          - name: ARGS
            value: ${APP_ARGS}
          image: ${APPLICATION_NAME}
          name: ${APPLICATION_NAME}
          ports:
          - containerPort: 8080
    triggers:
    - imageChangeParams:
        automatic: true
        containerNames:
        - ${APPLICATION_NAME}
        from:
          kind: ImageStreamTag
          name: ${APPLICATION_NAME}:latest
      type: ImageChange
    - type: ConfigChange
```
### Services
```yaml
- apiVersion: v1
  kind: Service
  metadata:
    annotations:
      description: Exposes and load balances the application pods
    name: ${APPLICATION_NAME}
  spec:
    ports:
    - name: web
      port: 8080
      targetPort: 8080
    selector:
      name: ${APPLICATION_NAME}
```
### Route
```yaml
- apiVersion: v1
  id: ${APPLICATION_NAME}
  kind: Route
  metadata:
    annotations:
      description: Route for application's http service.
    labels:
      application: ${APPLICATION_NAME}
    name: ${APPLICATION_NAME}
  spec:
    host: ${APPLICATION_DOMAIN}
    to:
      name: ${APPLICATION_NAME}
```
### Parameters
```yaml
parameters:
- description: The URL of the repository with your Golang S2I builder Dockerfile
  name: BUILDER_SOURCE_REPOSITORY_URL
  value: https://github.com/rhtps/golang-s2i.git
- description: Set this to a branch name, tag or other ref of your repository if you
    are not using the default branch
  name: BUILDER_SOURCE_REPOSITORY_REF
- description: Set this to the relative path to your project if it is not in the root
    of your repository
  name: BUILDER_CONTEXT_DIR
- description: A secret string used to configure the GitHub webhook for the builder repo
  from: '[a-zA-Z0-9]{40}'
  generate: expression
  name: BUILDER_GITHUB_WEBHOOK_SECRET
- description: The URL of the repository with your Golang application code
  name: APP_SOURCE_REPOSITORY_URL
- description: Set this to a branch name, tag or other ref of your repository if you
    are not using the default branch
  name: APP_SOURCE_REPOSITORY_REF
- description: Set this to the relative path to your project if it is not in the root
    of your repository
  name: APP_CONTEXT_DIR
- description: A secret string used to configure the GitHub webhook for the app repo
  from: '[a-zA-Z0-9]{40}'
  generate: expression
  name: APP_GITHUB_WEBHOOK_SECRET
- description: The name for the application.
  name: APPLICATION_NAME
  required: true
  value: golang-app
- description: 'Custom hostname for service routes.  Leave blank for default hostname,
    e.g.: <application-name>.<project>.<default-domain-suffix>'
  name: APPLICATION_DOMAIN
- description: Command line arguments to provide to the Golang application
  name: APP_ARGS
```
### Final
Let's create the template
```shell
[vagrant@rhel-cdk golang-s2i]$ cd openshift

[vagrant@rhel-cdk openshift]$ oc login -u openshift-dev

[vagrant@rhel-cdk openshift]$ oc new-project gochat

[vagrant@rhel-cdk openshift]$ oc create -f golang.yaml
```
Application arguments (APP_ARGS)
```shell
-host=0.0.0.0:8080 -callBackHost=10.1.2.2:8080 -templatePath=/opt/app-root/gopath/src/github.com/rhtps/gochat/templates -avatarPath=/opt/app-root/gopath/src/github.com/rhtps/gochat/avatars
```
