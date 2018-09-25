---
layout: single
title: "Docker for Java Developers"
date: '2018-09-16'
last_modified_at: 2018-09-16
comments: true
categories:
  - Docker, Java
tags:
  - docker, java, maven
toc: true
toc_label: "on this page"
toc_icon: "list"
---

links
https://github.com/arun-gupta/docker-for-java/tree/master/chapter2
https://github.com/spotify/docker-client
https://github.com/dockerfile/java
https://github.com/fabric8io/docker-maven-plugin
https://dmp.fabric8.io/

Communication to the daemon in OSX works via socket
docker run -d -v /var/run/docker.sock:/var/run/docker.sock -p 127.0.0.1:1234:1234 bobrik/socat TCP-LISTEN:1234,fork UNIX-CONNECT:/var/run/docker.sock
export DOCKER_HOST=tcp://localhost:1234

or 
.withDockerHost("unix:///var/run/docker.sock")

docker registry???
"https://index.docker.io/v1/ ??? server address to reach the docker engine which needs credentials to provide an answer
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
The Docker Engine can keep user credentials in an external credentials store, such as the native keychain of the operating system. 
Using an external store is more secure than storing credentials in the Docker configuration file.
You need to specify the credentials store in $HOME/.docker/config.json to tell the docker engine to use it. 

docker logs
https://docs.docker.com/config/daemon/#read-the-logs
