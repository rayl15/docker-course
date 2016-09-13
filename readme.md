# Deep Dive into Docker
These are course notes I'm taking to remember things.

- [Learning The Basics of Docker](#)
    - [Introduction to Docker](#)
    - [Containers Vs VirtualMachines](#)
    - [Docker Architecture](#)
    - [Docker Installation](#)
    - [Creating our First Image](#creating-our-first-image)
- [The Dockerfile, Builds and Network Configuration](#)
    - [Dockerfile Directives: User and Run](#)
    - [Dockerfile Directives: RUN Order of Execution](#)
    - [Dockerfile Directives: ENV](#)
    - [Dockerfile Directives: CMD Vs Run](#)
    - [Dockerfile Directives: ENTRYPOINT](#)
    - [Dockerfile Directives: EXPOSE](#)
    - [Container Volume Management](#)
    - [Docker Network: List and Inspect](#)
    - [Docker Network: Create and Remove](#)
    - [Docker Network: Assign to Containers](#)
- [Docker Commands and Structures](#)
    - [Inspect Container Processes](#)
    - [Previous Container Management](#)
    - [Controlling Port Exposure on Containers](#)
    - [Naming our Containers](#)
    - [Docker Events](#)
    - [Managing and Removing Base Images](#)
    - [Saving and Loading Docker Images](#)
    - [Image History](#)
    - [Taking Control of Our Tags](#)
    - [Pushing to DockerHub](#)
- [Integration and use Cases](#)
    - [Building a Web Farm for Development and Testing - Prerequisites](#)
    - [Building a Web Farm for Development and Testing - Part 1](#)
    - [Building a Web Farm for Development and Testing - Part 2](#)
    - [Building a Web Farm for Development and Testing - Part 3](#)
    - [Building a Web Farm for Development and Testing - Part 4](#)
    - [Integrating Custom Network in Your Docker Containers](#)
    - [Testing Version Compatibility - Using Tomcat and Java - Prerequisites](#)
    - [Testing Version Compatibility - Using Tomcat and Java - Part 1](#)
    - [Testing Version Compatibility - Using Tomcat and Java - Part 2](#)
    - [Testing Version Compatibility - Using Tomcat and Java - Part 3](#)


# Learning the Basics of Docker
---

## Introduction to Docker
Skip

## Containers Vs VirtualMachines
Skip

## Docker Architecture
Skip

## Docker Installation
Skip

## Creating our First Image
See our images
```
docker images
```
Check Version
```
docker version
```

Let's us know API version and google-go version for docker daemon.
```
docker info
```

- Locations
    - **Where Container Details are Located**
        - `var/lib/docker` is where all `images/containers` are stored
        - cd `var/lib/docker/containers/<hex>`
    - **Where Images are Located**
        - ls `/var/lib/docker/image/aufs/imagedb/content/sha256`

Whats Running
```
docker ps
```

What has run (With no name passed, it makes up a name)
```
docker ps -a
```

Refer to a Container by `CONTAINER_ID` or `NAME`

#### Pull an image
```
docker pull ubuntu (would be latest)
docker pull ubuntu:trusty (Or any tag listed in dockerhub)
```

"Container Layers build the docker image"

#### Launch container
(i = interactive, -t = attach to terminal)
This will launch the container, but not keep it running once we exit
```
docker run -it ubuntu:xenial /bin/bash
```

```
docker ps -a
```

Grab name name of what we ran, eg 'adoring_einstein'

```
docker restart adoring_einstein
docker ps
docker attach adoring_einstein  (Logs us in)
```

#### Keep the container running in Background
(disconnected/daemonized)

```
docker run -itd ubuntu:xenial /bin/bash
```

#### Run another Instance

```
docker run -itd ubuntu:xenial /bin/bash
```

#### Info about the Base image
```
docker inspect ubuntu:xenial
docker ps  (See the different names)
```

#### Info about Container
```
docker ps -a
docker inspect compassionate_bhaskara
docker inspect compassionate_bhaskara | grep IP
```

#### Going into container

```
docker attach compassionate_bhaskara
<Enter>
```

#### Stopping Containers
```
docker stop <name>
docker ps -a
```

#### Searching for containers
```
docker search ruby
docker search training/sinatra
```

#### Instance Examples
```
docker pull training/sinatra
docker run -it training/sinatra /bin/bash
gem
gem list --local
```

#### Create a file in this instance
```
cd /root
echo "Testing" > test.txt
exit
```

#### Notice new instances don't have our old items
```
docker run -it training/sinatra /bin/bash
ls /root
```

#### We can reboot our old instance
```
docker ps -a
docker restart sad_bohr
docker attach sad_bohr
ls /root         ;(Our test.txt remains)
```

# Packaging a custom container
Instantiate
```
docker ps -a
docker run -it ubuntu:xenial /bin/bash
cd /root
echo "This is version 1 of our custom image" > image_version.txt
apt-get update
apt-get install telnet openssh-server

adduser test

which sshd
which telnet

cat /etc/group | grep test

docker ps -a

docker restart compassionate_cori
docker attach compassionate_cori

docker commit -m "Already installed SSH and created test user" -a "imboyus" compassionate_cori imboyus/ubuntusshd:v1

docker images

docker run -it imboyus/ubuntusshd:v1 /bin/bash
```

#### Build a Dockerfile from box
```
vim Dockerfile
```

**Dockerfile**
```
# This is a ubuntu image with SSH already installed
FROM ubuntu:xenial
MAINTAINER imboyus <imboyus@gmail.com>
RUN apt-get update
RUN apt-get install -y telnet openssh-server
```

```
docker build -t="imboyus/ubuntusshdonly:v2" .
docker images
docker run -t imboyus/ubuntusshdonly:v2 /bin/bash
```

# Running Container Commands with Docker
```
docker images
docker run -it ubuntu:xenial /bin/bash
```

```
ps
```
Processes (`ps`) are contained within the docker container, but the perfomance (`top`)
shows the host system performance.

#### See the logs from a container (Running or not running)
```
docker ps -a
docker restart cocky_archimedes
docker ps
docker logs cocky_archimedes
```

#### Run commands on a running container
Exec only works on running containers nor will it auto-start it.
```
docker exec cocky_archimedes /bin/cat /etc/profile

How to know for sure:
---------------------
docker attach cocky_archimedes
vi /etc/profile     (edit something)

docker restart cocky_archimedes
docker exec cocky_archimedes /bin/cat /etc/profile
```

```
docker run ubuntu:xenial /bin/echo "Hello from this Container"
ps -a  (Notice this creates an instance, though not running)

; the name, eg: admiring_meitner

docker logs admiring_meitner
```

#### Containerize

```
docker run -d ubuntu:xenial /bin/bash -c "while true; do echo  HELLO; sleep 1; done"
docker ps
docker logs lonely_bose
docker logs lonely_bose | wc -l
docker stop lonely_bose
```

# Exposing Container with Port Redirects
A port must be exposed to the underlying Host.

```
; We know this uses port 80

docker pull nginx:latest
```

```
docker run -d nginx:latest
docker ps
docker inspect zen_goldberg

Look at the IP Address, eg: 172.17.0.2
```

Install Text Web browser
```
apt-get install elinks
```

See it running
```
elinks http://172.17.0.2
elinks http://localhost (does not find it on underlying host)
```

Run in Daemon mode redirect my local port order to the remote port
```
docker ps
docker stop zen_goldberg

eg: docker run -d -p <local>:<container/remote> nginx:latest

docker run -d -p 8080:80 nginx:latest
docker ps

elinks http://172.17.0.2  (Works)
elinks http://localhost:8080  (Its redirecting traffic from docker daemon)
```

# The Dockerfile, Builds and Network Configuration
---

## Dockerfile Directives: USER and RUN
Dir: `buids/01_RunAsUser`
```
docker pull centos:latest
```

*Dockerfile*

FROM must be the first directive

```
# Dockerfile based on the latest CentOS 7 image - non-privileged user entry
FROM centos:latest
MAINTAINER imboyus@gmail.com

RUN useradd -ms /bin/bash user
USER user
```

docker build -t centos7/nonroot:v1 .
docker run -it centos centos7/nonroot:v1 /bin/bash

_note: When rebuilding an image, it only changes additions and keeps the cache from already existing command builds_

Since a user can't login, we login as user:
```
docker ps -a
docker start cocky_boyd
docker exec -u 0 -it cocky_boyd /bin/bash
```

## Dockerfile Directives: RUN Order of Execution
Dir: `buids/02_CustomMessage`

- Order of Execution
    - `FROM`
    - `MAINTAINER`
    - `RUN`
        - * _If you make a `USER` AFTER this, everything will run as THAT user_

- Important
    - `RUN`: execute at build time, becomes part of the base image.
    - `CMD`: Runs when a container is instantiated, eg: Run an application.

```
docker build -t centos7/config:v1 .
docker run -it centos7/config:v1 /bin/bash
cat /etc/exports.list
```

## Dockerfile Directives: ENV
Dir: `buids/03_JavaInstall`

**When doing updates or installs always do `apt-get update -y` or `yum update -y`**

```
Docker build -t centos7/java8:v1 .      (You'll see red, that's ok)
```

Check the ENV is set for Java, though limited to one user
```
docker images
docker run -it centos7/java8:v1 /bin/bash
env
```

Control System-wide variables
```
Dockerfile:
ENV JAVA_BIN /usr/java/jdk1.8.0/jre
```

```
docker run -it centos7/java8:v2 /bin/bash
env     (Now we see JAVA_HOME global)
```

## Dockerfile Directives: CMD vs RUN
Dir: `buids/04_EchoServer`

- Differences
    - `RUN`: execute at build time, becomes part of the **base image**.
    - `CMD`: Runs when a container is instantiated, or container starts. eg: Run an application. **not part of the build process**

```
docker build -t centos7/echo:v1 .
docker images
docker run centos7/echo:v1  ; Should see an echo from the CMD
```

## Dockerfile Directives: ENTRYPOINT
Dir: `buids/05_Entry`

Sets default application used everytime a container is created even if you ask it to do something else.

```
docker build -t centos7/entry:v1 .
docker images

; Test these
docker run centos7/entry:v1
docker run centos7/echo:v1  /bin/bash echo "See me?"      ; Yes (Notice this is /echo)
docker run centos7/entry:v1  /bin/bash echo "See me?"     ; No
docker run centos7/entry:v1                               ; Outputs default CMD
```

- Different between ENTRYPOINT and CMD
    - `ENTRYPOINT` - Runs no matter what
    - `CMD` - CAN be overwritten for a command, eg: in an echo example above

## Dockerfile Directives: EXPOSE
Dir: `buids/06_ApacheInstallation`

- Commands
    - `-P` is to run/remap any ports in the container

```
docker build -t centos7/apache:v1 .
docker images

# Run container as daemon so it doesn't exit and think it's done

docker run -d --name apacheweb1 centos7/apache:v1
docker ps
docker inspect apacheweb1 | grep IPAdd
```
__Dont worry about the red GPG keys__

See it all works
```
elinks http://172.17.0.3
docker exec apacheweb1 /bin/cat /var/www/html/index.html
docker ps    (Container still running)
```

No Ports Exposed, so:
```
docker stop apacheweb1

docker run -d --name apacheweb2 -P centos7/apache:v1
docker ps    (no ports exposed still)
docker stop apacheweb2
```

Manually Remap ports outside of dockerfile
````
docker run -d --name apacheweb3 -p 8080:80 centos7/apache:v1
docker ps    (ports are exposed)
elinks http://localhost:8080
```

Dockerfile, add:
```
EXPOSE 80
```

Rebuild and check it out
```
docker build -t centos7/apache:v1 .
docker run -d --name apacheweb4 -P centos7/apache:v1
docker ps
```

## Container Volume Management
Dir: `buids/07_MyHostDir`

- Volumes:
    - Mounts
    - File Systems
    - `-v` Flag
        - 1: Create a mount outside containers FS, exposes whatever we put in there to underlying host,
        - 3: Gives ability to provide direct access from host mount to container (eg vagrant, no NFS needed)

```
docker run -it --name voltest1 -v /mydata centos:latest /bin/bash
df -h  (See a /mydata folder which is outside the normal container)

echo "this is a test container file" > /mydata/mytext.txt
exit
```

Where is `/mydata/mytext.txt?
```
docker inspect voltest1 | grep Source
cd /var/lib/docker/volumes/<hash-code-here>/_data
cat mytext.txt

# This will be in the mount on a base image:
echo "this is from me 2" > host.txt
```

Sync <local-path>:<container-path>
```
docker run -it --name voltest2 -v ~/docker-course/builds/MyHostDir:/mydata centos:latest /bin/bash
```

You can use the `VOLUMES` directive in `Dockerfile`, but local files may not be available
for containers, so it may not always be the best for mass distribution.


## Docker Network: List and Inspect

You can see `docker0` which is a bridge default on `172.17.0.1` below, and it will
grab an available IP from docker0:
```
ifconfig    ( look at docker0)
```

Current networks associated with current host:
```
; These are always unique identifiers
docker network ls

; If you need to see the full hash
docker network ls --no-trunc
```

Get Details about a network:
```
docker network inspect bridge

Outputs:

[
    {
        "Name": "bridge",
        "Id": "f49ee9618b80a2423f7ff7725ab335129b59abc441731161b6a0f5db6dd67d80",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            ...
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

## Docker Network: Create and Remove
```
Right:
man docker-network-create (Use this)

Wrong:
man docker network create (Does not work)
```

Create a network with new subnet to use with a bridge for local adapters:

__Most of the time you create subnets for locally routed interal items.__
```
docker network create --subnet 10.1.0.0/24 --gateway 10.1.0.1 mybridge01
docker network ls

ifconfig    (br-<hash> matches the docker network ls ID)
```

Remove a network, be careful to not remove the default 3 networks. You cannot undo it,
and if you do you are better off re-installing docker completely.

```
docker network rm mybridge01
docker network ls
```

Recreate
```
docker network create --subnet 10.1.0.0/24 --gateway 10.1.0.1 mybridge01
docker network inspect mybridge01
docker network rm mybridge01
```

## Docker Network: Assign to Containers

Control how IP's are assigned to our containers.

Class B network with subnet having large range of addresses, and tell the host
only to assign IP's to containers from a subset range for that network.

(About 190 IP's available)

- `--gateway` is how to route to pass the traffic from the underlying host
- `--driver`
    - `bridge` is for a simple setup
    - `overlay` is for a multi-cluster

```
ip/24 = 1-254 (Class C address, last octet)

docker network create --subnet 10.1.0.0/16 --gateway 10.1.0.1 --ip-range=10.1.4.0/24 --driver=bridge --label=host4network bridge04

docker network ls
docker network inspect bridge04
ifconfig
```

Create a Docker Container and use Network
```
docker run -it --name nettest1 --net bridge04 centos:latest /bin/bash

yum update
yum install -y net-tools
ifconfig   (We have 10.1.4.0)
netstat -rn
ping google.com
cat /etc/resolv.conf

exit
```

Static IP's only work on user created networks.
```
docker run -it --name nettest2 --net bridge04 --ip 10.1.4.100 centos:latest /bin/bash
yum update
yum install -y net-tools
ifconfig    (Should show inet 10.1.4.100)
netstat -rn
```

In a new Terminal with the container running we can do:
```
docker inspect nettest2 | grep IPAddr
ping 10.1.4.100
```

Pre-Register DNS with Static IP's, which helps with provisioning and such by using
a static IP.


# Docker Commands and Structures
---

## Inspect Container Processes
Coming
## Previous Container Management
Coming
## Controlling Port Exposure on Containers
Coming
## Naming our Containers
Coming
## Docker Events
Coming
## Managing and Removing Base Images
Coming
## Saving and Loading Docker Images
Coming
## Image History
Coming
## Taking Control of Our Tags
Coming
## Pushing to DockerHub
Coming

# Integration and use Cases
---

## Building a Web Farm for Development and Testing - Prerequisites
Coming
## Building a Web Farm for Development and Testing - Part 1
Coming
## Building a Web Farm for Development and Testing - Part 2
Coming
## Building a Web Farm for Development and Testing - Part 3
Coming
## Building a Web Farm for Development and Testing - Part 4
Coming
## Integrating Custom Network in Your Docker Containers
Coming
## Testing Version Compatibility - Using Tomcat and Java - Prerequisites
Coming
## Testing Version Compatibility - Using Tomcat and Java - Part 1
Coming
## Testing Version Compatibility - Using Tomcat and Java - Part 2
Coming
## Testing Version Compatibility - Using Tomcat and Java - Part 3
Coming









