# Intro to Docker

Ron Scott

https://github.com/ronaldscott

---

## Intro to Containers

---

### Containers

* run
* package
* deploy

...software in isolation

A process running in a container is sandboxed from the host OS.

---

### Is a container a kind of virtual machine?

Containers are not virtual machines, but...

thinking of them as VMs provides a useful comparison.

The functionality is similar, and many of the concepts translate well.

---

### Is a container a kind of virtual machine?

Most of the time, you can think of a container as a tiny, lightweight VM for just one process

(_even though that's not strictly accurate_)

---

### VMs vs Containers

![VMs vs. Containers](https://images.idgesg.net/images/article/2017/06/virtualmachines-vs-containers-100727624-large.jpg)

---

### Operating System Features

A combination of OS features makes containers possible.

---

### cgroups

> cgroups (abbreviated from control groups) is a Linux kernel feature that limits, accounts for, and isolates the resource usage (CPU, memory, disk I/O, network etc.) of a collection of processes. 

-- [wikipedia: cgroups](https://en.wikipedia.org/wiki/Cgroups)

---

### Namespaces

> Namespaces are a feature of the Linux kernel that isolates and virtualizes system resources of a collection of processes. Examples of resources that can be virtualized include process IDs, hostnames, user IDs, network access, interprocess communication, and filesystems.

--[wikipedia: Linux namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)

---

#### Union Mounting

> In computer operating systems, union mounting is a way of combining multiple directories into one that appears to contain their combined contents. Union mounting is supported in Linux, BSD and several of its successors, and Plan 9, with similar but subtly different behavior.

--[wikipedia: Union mount](https://en.wikipedia.org/wiki/Union_mount)

---

#### Union Mounting

![Union Mounting](http://cecs.wright.edu/~pmateti/Courses/3900/Lectures/Internals/Figures/mount-bind.png)

---

#### Docker is Just Tooling

---

Anyone can use basic Linux features to implement containers. For example, [bocker](https://github.com/p8952/bocker) is a basic Docker clone, written in ~100 lines of Bash, using built-in Linux features and a couple of extra utilities like curl.

---

Container tooling adds:

* helper functions
* repeatability
* automation
* infrastructure

---

Many different containerization toolsets, including:

* Canonical's **LXC** and its successor **LXD**
* **Docker**, an open-source product sponsored by Docker, Inc.
* **rkt** (rocket), a community attempt to build a more open, community-driven container engine

---

The rest of this talk is about Docker, but!

Remember that Docker is only _one take_ on containerization tooling.

Other valid alternatives exist.

---

#### Docker fundamentals

Docker provides a lot of higher-level concepts, APIs, apps, and infrastructure on top of the basic virtualization capabilities of Linux.

---

#### Image

> An image is a read-only template with instructions for creating a Docker container. Often, an image is based on another image, with some additional customization.

--[docker](https://docs.docker.com/engine/docker-overview/#docker-objects)

---

#### Images

* _Built_ from a set of instructions called a _Dockerfile_
* Not checked into source control; _Dockerfiles_ are
* Repeatable, by cloning the Dockerfile and building a copy of the image

---

#### Sample Dockerfile

```dockerfile
FROM debian:jessie

MAINTAINER Daniel Alan Miller <dalanmiller@rethinkdb.com>

# Add the RethinkDB repository and public key
# "RethinkDB Packaging <packaging@rethinkdb.com>" http://download.rethinkdb.com/apt/pubkey.gpg
RUN apt-key adv --keyserver keys.gnupg.net --recv-keys 3B87619DF812A63A8C1005C30742918E5C8DA04A
RUN echo "deb http://download.rethinkdb.com/apt jessie main" > /etc/apt/sources.list.d/rethinkdb.list

ENV RETHINKDB_PACKAGE_VERSION 2.3.6~0jessie

RUN apt-get update \
	&& apt-get install -y rethinkdb=$RETHINKDB_PACKAGE_VERSION \
	&& rm -rf /var/lib/apt/lists/*

VOLUME ["/data"]

WORKDIR /data

CMD ["rethinkdb", "--bind", "all"]

#   process cluster webui
EXPOSE 28015 29015 8080
```

---

#### Container

> A container is a runnable instance of an image. You can create, start, stop, move, or delete a container using the Docker API or CLI. You can connect a container to one or more networks, attach storage to it, or even create a new image based on its current state.

--[docker](https://docs.docker.com/engine/docker-overview/#docker-objects)

---

#### Docker Engine

Client/server application

* A daemon process (`dockerd`) that runs Docker containers as services
* A REST API that clients can use to manipulate `dockerd`
* A client-side CLI (`docker`) that interacts with `dockerd` through the REST API

---

#### Docker Architecture

![Docker Architecture](https://docs.docker.com/engine/images/architecture.svg)

---

#### Docker Registry

> The Docker Registry is a stateless, highly scalable server side application that stores and lets you distribute prebuilt Docker images. The Registry is open-source, under the permissive Apache license. 

--[docker](https://docs.docker.com/registry/)

---

#### Docker Hub

The [Docker Hub](https://hub.docker.com/) is the official image registry. Similar to a public package registry, like npm or NuGet.

---

#### Private Registry

Anyone can self-host a private Docker registry. Many organizations do so. This is analogous to hosting a private package registry, like NuGet.

---

### Docker on Windows

Docker, Inc. also maintains a Windows-native version of Docker.

---

### Linux Containers on Windows

Docker for Windows can use Hyper-V to run a fully virtualized Linux environment, which, in turn, hosts containers.

---

### Linux Containers caveat

Enabling Hyper-V disables all other VM products. You can't use VirtualBox or VMWare.

---

#### Windows containers on Windows

Windows 10 (Professional or Enterprise) and Windows Server 2016 offer _native support_ for Windows containers. Windows containers are a Windows-native container implementation.

---

#### Demo: Development workflow

<!-- Build and run some code in a container --

---

#### Demo: Appliance hosting

<!-- Pull some common appliance and run it -->

---

#### Demo: AWS ECS

<!-- Push first container to ECS and run them there -->

---

#### Uses for Docker

---

* **Environment consistency**: Run the same code in an repeatable, identical environment, from dev straight up to prod
* **Infrastructure as code**: All configuration and setup for a container are repeatable and stored in a repo
* **Simplified dev environment**: run server dependencies as appliances instead of as native installations in the host environment
* **Ease dependency hell**: Different containers can have their own library versions on the same host environment

---

## FIN