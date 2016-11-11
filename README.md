
# Source-to-Image Workshop Content

This document contains the code/configuration snippets corresponding to the workshop slides.

<!-- TOC depthFrom:2 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Pre Docker](#pre-docker)
	- [Pre Docker - Step 1](#pre-docker-step-1)
	- [Pre Docker - Step 2](#pre-docker-step-2)
	- [Pre Docker - Step 3](#pre-docker-step-3)
	- [Pre Docker - Step 4](#pre-docker-step-4)
	- [Pre Docker - Step 5](#pre-docker-step-5)
- [Option 1 - Dockerfile](#option-1-dockerfile)
	- [Dockerfile - Step 1](#dockerfile-step-1)
	- [Dockerfile - Step 2](#dockerfile-step-2)
	- [Dockerfile - Step 3](#dockerfile-step-3)
	- [Dockerfile - Step 4](#dockerfile-step-4)
	- [Dockerfile - Step 5](#dockerfile-step-5)
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
- [Automation in OpenShift](#automation-in-openshift)
	- [Automation - Step 1](#automation-step-1)
	- [Automation - Step 2](#automation-step-2)
	- [Automation - Step 3](#automation-step-3)
	- [Automation - Step 4](#automation-step-4)
	- [Automation - Step 5](#automation-step-5)

<!-- /TOC -->
<a name="pre-docker"></a>
## Pre Docker
<a name="pre-docker-step-1"></a>
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
<a name="pre-docker-step-2"></a>
### Pre Docker - Step 2
```shell
# Obtain golang tooling
[student@localhost ~]$ curl -k -o go1.6.2.linux-amd64.tar.gz https://storage.googleapis.com/golang/go1.6.2.linux-amd64.tar.gz

[student@localhost ~]$ tar -xvf go1.6.2.linux-amd64.tar.gz
```
<a name="pre-docker-step-3"></a>
### Pre Docker - Step 3
```shell
# Obtain Canonical Bazaar
[student@localhost ~]$ curl -k -L -o bzr.tar.gz https://launchpad.net/bzr/2.7/2.7.0/+download/bzr-2.7.0.tar.gz

[student@localhost ~]$ tar -xvf bzr.tar.gz

```
<a name="pre-docker-step-4"></a>
### Pre Docker - Step 4
```shell
#Download and build the gochat application
[student@localhost ~]$ cd ~

[student@localhost ~]$ go get -insecure github.com/rhtps/gochat

[student@localhost ~]$ go install github.com/rhtps/gochat
```
<a name="pre-docker-step-5"></a>
### Pre Docker - Step 5
```shell
[student@localhost ~]$ gochat -host=localhost:8080 -callBackHost=http://localhost:8080 -templatePath=$GOPATH/src/github.com/rhtps/gochat/templates/ -avatarPath=$GOPATH/src/github.com/rhtps/gochat/avatars -htpasswdPath=$GOPATH/src/github.com/rhtps/gochat/htpasswd
```
<a name="option-1-dockerfile"></a>
## Option 1 - Dockerfile
<a name="dockerfile-step-1"></a>
### Dockerfile - Step 1
Start the CDK and instal CLI tools.
```shell
[student@localhost ~]$ cd ~/cdk/components/rhel/rhel-ose/

[student@localhost rhel-ose]$ vagrant up

#==> default: Registering box with vagrant-registration...
#    default: Would you like to register the system now (default: yes)? [y|n] n

[student@localhost rhel-ose]$ eval "$(vagrant service-manager env docker)"

[student@localhost rhel-ose]$ eval "$(VAGRANT_NO_COLOR=1 vagrant service-manager install-cli docker | tr -d '\r')"

[student@localhost rhel-ose]$ eval "$(VAGRANT_NO_COLOR=1 vagrant service-manager install-cli openshift | tr -d '\r')"
```
<a name="dockerfile-step-2"></a>
### Dockerfile - Step 2
```shell
[student@localhost ~]$ cd ~

[student@localhost ~]$ git clone https://github.com/rhtps/gochat-docker.git

[student@localhost ~]$ less gochat-docker/Dockerfile
```
<a name="dockerfile-step-3"></a>
### Dockerfile - Step 3
```shell
# Build the image
[student@localhost ~]$ cd ~/gochat-docker

[student@localhost gochat-docker]$ docker build -t gochat-docker .
```
<a name="dockerfile-step-4"></a>
### Dockerfile - Step 4
```shell
# Start the gochat container
[student@localhost ~]$ docker run -d -p 8080:8080 --name gochat gochat-docker -host=0.0.0.0:8080 -callBackHost=http://0.0.0.0:8080 -templatePath=/opt/gopath/src/github.com/rhtps/gochat/templates -avatarPath=/opt/gopath/src/github.com/rhtps/gochat/avatars -htpasswdPath=/opt/gopath/src/github.com/rhtps/gochat/htpasswd

[student@localhost ~]$ docker ps
```
<a name="dockerfile-step-5"></a>
### Dockerfile - Step 5
```shell
[student@localhost ~]$ docker stop gochat

[student@localhost ~]$ sudo docker rm gochat
```
---
<a name="option-2-source-to-image-s2i"></a>
## Option 2 - Source-to-Image (S2I)
<a name="s2i-step-1"></a>
### S2I - Step 1
```shell
[student@localhost ~]$ go get github.com/openshift/source-to-image

[student@localhost ~]$ cd $GOPATH/src/github.com/openshift/source-to-image

[student@localhost source-to-image]$ export PATH=$PATH:$GOPATH/src/github.com/openshift/source-to-image/_output/local/bin/linux/amd64/

[student@localhost source-to-image]$ hack/build-go.sh
```
<a name="s2i-step-2"></a>
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
<a name="s2i-step-3"></a>
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
<a name="s2i-step-4"></a>
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
<a name="s2i-step-5"></a>
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
<a name="s2i-step-6"></a>
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
<a name="s2i-step-7"></a>
### S2I - Step 7
```shell
[student@localhost ~]$ cat /dev/null > ~/golang-s2i/.s2i/bin/usage

[student@localhost ~]$ vi ~/golang-s2i/.s2i/bin/usage
```
<a name="s2i-step-8"></a>
### S2I - Step 8
```shell
[student@localhost ~]$ cd ~/golang-s2i/

[student@localhost golang-s2i]$ docker build -t golang-s2i .
```
<a name="s2i-step-9"></a>
### S2I - Step 9
```shell
# s2i build <source location> <builder image> [<tag>] [flags]

[student@localhost golang-s2i]$ s2i build https://github.com/rhtps/gochat.git golang-s2i gochat
```
<a name="s2i-step-10"></a>
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
<a name="automation-in-openshift"></a>
## Automation in OpenShift
<a name="automation-step-1"></a>
### Automation - Step 1
```shell
[student@localhost ~]$ cd ~

[student@localhost ~]$ mkdir ~/openshift

[student@localhost ~]$ cd ~/openshift

[student@localhost ~]$ vi golang.yaml
```
<a name="automation-step-2"></a>
### Automation - Step 2
Add the following to the template.
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
- apiVersion: v1
  kind: ImageStream
  metadata:
    annotations:
      description: Keeps track of changes in the application image
    name: ${APPLICATION_NAME}
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
          name: golang-s2i:latest
      type: Source
    triggers:
    - type: ImageChange
    - type: ConfigChange
    - github:
        secret: ${APP_GITHUB_WEBHOOK_SECRET}
      type: GitHub
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
parameters:
- description: The URL of the repository with your Golang application code
  name: APP_SOURCE_REPOSITORY_URL
  displayName: Application Source URL
- description: Set this to a branch name, tag or other ref of your repository if you
    are not using the default branch
  name: APP_SOURCE_REPOSITORY_REF
  displayName: Application Source Branch
- description: Set this to the relative path to your project if it is not in the root
    of your repository
  name: APP_CONTEXT_DIR
  displayName: Application Source Directory
- description: A secret string used to configure the GitHub webhook for the app repo
  from: '[a-zA-Z0-9]{40}'
  generate: expression
  name: APP_GITHUB_WEBHOOK_SECRET
  displayName: Application GitHub Web Hook Secret
- description: The name for the application.
  name: APPLICATION_NAME
  required: true
  value: golang-app
  displayName: Application Name
- description: 'Custom hostname for service routes.  Leave blank for default hostname,
    e.g.: <application-name>.<project>.<default-domain-suffix>'
  name: APPLICATION_DOMAIN
  displayName: Application Domain
- description: Command line arguments to provide to the Golang application
  name: APP_ARGS
  displayName: Application Command Line Arguments
```
<a name="automation-step-3"></a>
### Automation - Step 3
Login to the integrated container registry
```shell
[student@localhost ~]$ oc login -u openshift-dev 10.1.2.2:8443

[student@localhost ~]$ oc whoami -t
#Copy this token!

[student@localhost ~]$ docker login -u openshift-dev -p (the token value) -e openshift-dev@local.host 172.30.124.37:5000
```
<a name="automation-step-4"></a>
### Automation - Step 4
Tag and push the builder image to integrated container registry.  We created this builder image in an earlier step.  Now we need to make it available to OCP.

```shell
[student@localhost ~]$ docker tag golang-s2i 172.30.124.37:5000/gochat/golang-s2i

[student@localhost ~]$ docker push 172.30.124.37:5000/gochat/golang-s2i
```
<a name="automation-step-5"></a>
### Automation - Step 5
Now we are going to deploy our application.

Application Source URL: **https://github.com/rhtps/gochat**

Application Source Branch: (blank)

Application Source Directory: (blank)

Application GitHub Web Hook Secret: (generated if empty)

Application Name: **gochat**

Application Domain: (blank)

Application Command Line Arguments:

```shell
-host=0.0.0.0:8080 -callBackHost=http://golang-app-gochat.rhel-cdk.10.1.2.2.xip.io -templatePath=/opt/app-root/gopath/src/github.com/rhtps/gochat/templates -avatarPath=/opt/app-root/gopath/src/github.com/rhtps/gochat/avatars -htpasswdPath=/opt/app-root/gopath/src/github.com/rhtps/gochat/htpasswd
```
