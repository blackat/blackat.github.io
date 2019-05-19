---
layout: single
title: "Docker Oracle database server container"
date: '2019-05-18'
last_modified_at: '2019-05-18'
comments: true
categories:
  - docker
tags:
  - [docker]
toc: true
toc_sticky: true
---

## Scenario exploration

Create a Oracle database container with a volume, create user and a schema, save the volume and reuse it in other container.

During development happens to create many times a database schema for different reasons:

- write persistent methods that alter the database schema or content
- recreate database schema from scratch to test the changes
- create the database for integration tests, at each test run a clean schema has to be created

Often the drop and the creation of the schema takes time, when this operation has to be done many time a day it becomes really time consuming. What about a simple script to revert the data of a database?

## Docker volumes

[By default][1] all files created inside a container are stored on a writable container layer:
- once the container is removed data is lost;
- not easy to share data with another container;
- container's writable layer is tightly coupled with the host machine hence really difficult to move data;
- the writable layer required a [storage driver][2] so an extra abstraction layer, instead data volumes write directly to the host filesystem.

*Volumes* are completely managed by Docker, they don't depend on the directory structure of the host machine.

## Oracle database server image

The Oracle database server image uses Docker volumes to store data, redo logs and so on. The data volume is mounted inside the container under `/ORCL`.

Start the container and inspect it:

```bash
$ docker system inspect oracle-12c
...
 "Mounts": [
            {
                "Type": "volume",
                "Name": "3defa65b36e7e227bc6f5ad4e43d4acf890b61994ac4b786b913e36500a04996",
                "Source": "/var/lib/docker/volumes/3defa65b36e7e227bc6f5ad4e43d4acf890b61994ac4b786b913e36500a04996/_data",
                "Destination": "/ORCL",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
```

typing:
```bash
$ docker volume ls
DRIVER              VOLUME NAME
local               3defa65b36e7e227bc6f5ad4e43d4acf890b61994ac4b786b913e36500a04996
```

there is data volume used by Oracle database server container.

## Create volumes

To bettre identify the volume used by the container, create a volume:

```bash
$ docker volume create oradata-volume
```

List volumes:

```bash
$ docker volume ls
DRIVER              VOLUME NAME
local               oradata-volume
```

Inspect a volume:

```bash
$ docker volume inspect volume-oradata
[
    {
        "CreatedAt": "2019-05-18T11:55:49Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/oradata-volume/_data",
        "Name": "oradata-volume",
        "Options": {},
        "Scope": "local"
    }
]
```

## Start a container with a volume

Let's consider the [Oracle database 18c Express Edition container][3]:

```bash
$ docker run -d -it --name oracle-12c -p 1521:1521 -v oradata-volume:/ORCL store/oracle/database-enterprise:12.2.0.1
```

So ask docker to create a container named `oracle-12c` mapping the port 1521 to the same port on the host machine, the same for the Oracle Enterprise Manager Express configured and reachable at https://localhost:5500/em/. Along with the container a volume created on the host is mapped to the one in the container.

Inspecting the container:
```bash
$ docker container inspect oralce-12c
"Mounts": [
            {
                "Type": "volume",
                "Name": "oradata-volume",
                "Source": "/var/lib/docker/volumes/oradata-volume/_data",
                "Destination": "/ORCL",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
```
verify the container uses the data volume as expected.

## Run SQLPlus inside container

Execute `bash` command inside the container

```bash
$ docker exec -it oracle-12c bash
```

Log into the database as system administrator:

```bash
sqlplus sys/Oradoc_db1@ORCLCDB as sysdba
```

Create a new user

```bash
SQL> alter session set "_ORACLE_SCRIPT"=true;
SQL> CREATE USER blackat IDENTIFIED BY password DEFAULT TABLESPACE users;
```

## Start a container with host directory

```bash
$ docker run -d -it --name oracle-12c -v oradata:/ORCL store/oracle/database-enterprise:12.2.0.1
oracle-12c
```

## Backup a container

Volumes are really useful to backup, restore and migrate data among containers.

The [Docker backup technique][5] is the following:

- create a new container mounting the `oracledata-volume` using the flag `--volume-from` to mount all the volumes from a given container;
- mount a local host system directory named `/backup`;
- pass a command to the container to tar the content of `ORCL` volume to a `backup.tar` file inside the `/backup` directory;
  - this directory is *shared* between the host and the container so the host will get the tar archive with the backup.

Create the folder `backup` on the host sytem, then use a Ubuntu container that is automatically removed after the work has been done.

```bash
$ docker run --rm --volumes-from oracle-12c -v backup:/backup ubuntu tar cvf /backup/backup.tar /ORCL
tar: Removing leading `/' from member names
/ORCL/
/ORCL/u04/
/ORCL/u04/app/
/ORCL/u04/app/oracle/
/ORCL/u04/app/oracle/redo/
/ORCL/u04/app/oracle/redo/redo002.log
/ORCL/u04/app/oracle/redo/redo003.log
/ORCL/u04/app/oracle/redo/redo001.log
/ORCL/u02/
/ORCL/u02/app/
...
```

During the running of the command the file list is logged to command line:

Once the command completes the backup of the volumne is availbale in the tar file.

### Something not working

If something not working as expected, an error message does not help to identify the cause of the porblem, run command step by step.

#### Run the container and verify the folder has been corretly mounted

```bash
$ docker run --name ubuntu -v backup:/backup ubuntu /bin/bash
```

the command completes inside the container, then verify the `backup` folder has been coreclty mounted running:

```bash
root@b2cda804f5fe:/# ls
backup  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

#### Run the container and verify the folder and volumes have been corretly mounted

```bash
$ docker run -it --name ubuntu --volumes-from oracle-12c -v backup:/backup ubuntu /bin/bash
```

the completes within the container, from the command line verify the `backup` folder and `ORCL` has been correctly mounted:

```bash
root@4c9bbda8cb8e:/# ls
ORCL  backup  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

Once identified and resolved the issues, exit the container and destroy it. Execute the the right command to produce the tar file.

## Database basic information

#### User and password

Connect to the database with user `sys` and password `Oradoc_db1`.

#### Connect from within the container

```bash
$ docker exec -it <oracle-db> bash -c "source /home/oracle/.bashrc; sqlplus /nolog"
```

#### Connect as sysdba

```bash
$ sqlplus sys/Oradoc_db1@ORCLCDB as sysdba
```

#### Custom database configuration

```bash
$ docker run -d -it --name <oracle-db> -P --env-file ora.conf store/oracle/database-enterprise:12.2.0.1
```

where `ora.conf` is the place to specify custom paramters such as:

- `DB_SID`: default is `ORCLCDB`;
- `DB_PDB`: default is `ORCLPDB1`;
- `DB_MEMORY`: default is 2GB;
- `DB_DOMAIN`: default it `localdomain`.

#### Change the dafult password

From within the container using SQLPlus

```bash
alter user sys identified by <new-password>;
```

#### Minimum requirements

Minimum requirements for the Oralce Enterprise container is 8GB of disk space and 2GB of memory.

#### Database logs

```bash
$ docker logs <oracle-db>
```

## Troubleshooting

On [Oracle GitHub page][3] there are bunch of scripts that help DevOps users to install, configure and setup the environment.

What is provided are build scripts and docker files based on an existing installation binary of Oracle database. If you run the command on the GitHub page you need to have built the image before, `docker pull` command does not download any images.

You could have the following error:

```bash
docker: Error response from daemon: pull access denied for oracle/database, repository does not exist or may require 'docker login'.
```

Even if you login into the Oracle container registry does not solve the issue, the image is not in the repository. Alternatively look for a Oracle enterprise Docker image on [Docker Hub](https://hub.docker.com), accept the `Terms of Service` and then run:

```bash
docker pull store/oracle/database-enterprise:12.2.0.1
```

## References

[1]: https://docs.docker.com/storage/
[2]: https://docs.docker.com/storage/storagedriver/
[3]: https://github.com/oracle/docker-images/tree/master/OracleDatabase/SingleInstance
[4]: https://container-registry.oracle.com/
[5]: https://docs.docker.com/storage/volumes/