---
layout: single
title: "Docker Development Workflow"
date: '2018-11-01'
last_modified_at: '2019-08-18'
comments: true
categories:
  - docker
tags:
  - [docker, container]
toc: true
toc_label: "On this page"
toc_icon: "list"
toc_sticky: true
---

## What is required

- CentOS 7 Docker image.
- WildFly
- OpenJDK
- Maven

Pull the Docker image of CentOS 7 or directly the WildFly images according to your customization level.

```bash
docker pull centos:7
```

or

```bash
docker pull jboss/wildfly
```

in this way you get an image based on CentOS 7, OpenJDK 11 and WildFly 11.
The default user is `jboss` and its working directory is `/opt/jboss` by default. The issue is that the `jboss` user is not in the group of sudoers hence it is not possible to install any new pakcage via `yum`.

Start the container in this way and then change the root password:

```bash
$ docker run -d -it --name dev -p 8080:8080 jboss/wildfly
284d107167f15be419d9e5b000e90e69bb9c033b5774b078d809878083569c44

$ docker  exec -it -u root dev /bin/bash
[root@284d107167f1 jboss]# passwd
Changing password for user root.
New password: *****
Retype new password: *****
passwd: all authentication tokens updated successfully.
```

Now install Maven:

```bash
$ yum install maven
```

follow the installation process and, in the end, verify Maven has been correctly installed typing:

```bash
[root@284d107167f1 jboss]# mvn -v
Apache Maven 3.0.5 (Red Hat 3.0.5-17)
Maven home: /usr/share/maven
Java version: 1.8.0_222, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64/jre
Default locale: en_US, platform encoding: ANSI_X3.4-1968
OS name: "linux", version: "4.9.184-linuxkit", arch: "amd64", family: "unix"
```

### Commit changes

Now commit the changes to create a new image from container changes

```bash
$ docker commit dev rdev
```

```bash
$ docker run -v "/Users/eugenio/Porjects/Pocs/externalize/src/dev/maven":/usr/share/maven
$ docker run -v "/Users/eugenio/Porjects/Pocs/externalize/src/dev/wildfly":/opt/jboss/wildfly
$ docker run -v "/Users/eugenio/Porjects/Pocs/externalize/":/home/jboss/externalize
```

and the working directory has to be set to `externalize`. Use `docker inspect rdev` to verify the mount has been correctly created.

As stated in the [Docker documentation](https://docs.docker.com/storage/bind-mounts/) _If you bind-mount into a non-empty directory on the container, the directory’s existing contents are obscured by the bind mount._