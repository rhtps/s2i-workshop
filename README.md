
# Source-to-Image Workshop Content

This document contains the code/configuration snippets corresponding to the workshop slides.

* [Source-to-Image Workshop Content](#source-to-image-workshop-content)
    * [Option 1 - Dockerfile](#option-1---dockerfile)
      * [Dockerfile - Step 1](#dockerfile---step-1)
      * [Dockerfile - Step 2](#dockerfile---step-2)
      * [Dockerfile - Step 3](#dockerfile---step-3)
      * [Dockerfile - Step 4](#dockerfile---step-4)
    * [Option 2 - Source-to-Image (S2I)](#option-2---source-to-image-s2i)
      * [S2I - Step 1](#s2i---step-1)
      * [S2I - Step 2](#s2i---step-2)
      * [S2I - Step 3](#s2i---step-3)
      * [S2I - Step 4](#s2i---step-4)
      * [S2I - Step 5](#s2i---step-5)
      * [S2I - Step 6](#s2i---step-6)
      * [S2I - Step 7](#s2i---step-7)
      * [S2I - Step 8](#s2i---step-8)
      * [S2I - Step 9](#s2i---step-9)
    * [Preparing for OpenShift](#preparing-for-openshift)
      * [Template](#template)
      * [ImageStreams](#imagestreams)
      * [BuildConfiguration](#buildconfiguration)
      * [DeploymentConfiguration](#deploymentconfiguration)
      * [Services](#services)
      * [Route](#route)
      * [Parameters](#parameters)
      * [Final](#final)


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
```
And ensure the file matches the following
```bash
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
```
And ensure the file matches the following
```bash
#!/bin/bash –e
exec $GOBIN/goexec $ARG
```
### S2I - Step 6
```bash
[ec2-user@ip-10-0-0-XX golang-s2i]$ vi .s2i/bin/save-artifacts
```
And ensure the file matches the following
```bash
#!/bin/bash –e

cd $GOPATH
tar cf - pkg bin src
```
### S2I - Step 7
```bash
[ec2-user@ip-10-0-0-XX golang-s2i]$ vi .s2i/bin/usage
```
### S2I - Step 8
```bash
[ec2-user@ip-10-0-0-XX golang-s2i]$ docker build –t golang-s2i .

# s2i build <source location> <builder image> [<tag>] [flags]

[ec2-user@ip-10-0-0-XX golang-s2i]$ s2i build https://github.com/rhtps/gochat.git golang-s2i gochat
```
### S2I - Step 9
```bash
[ec2-user@ip-10-0-0-XX golang-s2i]$ docker run -e "ARGS=-host=0.0.0.0:8080 -callBackHost=student01.s2i.rhtps.io:8080 -templatePath=/opt/go/src/github.com/rhtps/gochat/templates -avatarPath=/opt/go/src/github.com/rhtps/gochat/avatars" -p 8080:8080 -d --name chat gochat
```
When you are done
```bash
[ec2-user@ip-10-0-0-XX golang-s2i]$ docker stop chat

[ec2-user@ip-10-0-0-XX golang-s2i]$ docker rm chat
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

Log in to OpenShift from the command line
```bash
[ec2-user@ip-10-0-0-XX golang-s2i]$ oc login https://master.ose31.ravello.kenscloud.io:8443

[ec2-user@ip-10-0-0-XX golang-s2i]$ oc new-project golang

[ec2-user@ip-10-0-0-XX golang-s2i]$ oc create -f golang.yaml

```
Application arguments (APP_ARGS)
```bash
-host=0.0.0.0:8080 -callBackHost=student01.s2i.rhtps.io:8080 -templatePath=/opt/go/src/github.com/rhtps/gochat/templates -avatarPath=/opt/go/src/github.com/rhtps/gochat/avatars
```

###
