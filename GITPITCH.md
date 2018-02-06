# Intro to Docker

## Intro to Containers

Containers are a way of packaging, running, and deploying software in isolation. A process or collection of processes running within a container is sandboxed from the rest of the operating system. Containers are not virtual machines, but thinking of them as VMs provides a useful comparison, because the functionality is similar and many of the concepts translate well.

If a VM runs an entire operating system in a fully virtualized environment, a container runs just one or more virtualized processes, _without_ needing to incur the heavy performance penalty of spinning up an entire virtual machine.

![VMs vs. Containers][vmsvscontainers]

### Operating System Features

The combination of these related, but distinct, OS features is what makes containers possible. This is a non-comprehensive list; other kernel features are involved, especially related to security, but for the sake of time I will mention only the major features.

#### cgroups

cgroups (abbreviated from control groups) is a Linux kernel feature that limits, accounts for, and isolates the resource usage (CPU, memory, disk I/O, network etc.) of a collection of processes. [wikipedia](https://en.wikipedia.org/wiki/Cgroups)

#### Namespaces

Namespaces are a feature of the Linux kernel that isolates and virtualizes system resources of a collection of processes. Examples of resources that can be virtualized include process IDs, hostnames, user IDs, network access, interprocess communication, and filesystems. [wikipedia](https://en.wikipedia.org/wiki/Linux_namespaces)

#### Union Mounting

Union Mounting - In computer operating systems, union mounting is a way of combining multiple directories into one that appears to contain their combined contents.[1] Union mounting is supported in Linux, BSD and several of its successors, and Plan 9, with similar but subtly different behavior. [wikipedia](https://en.wikipedia.org/wiki/Union_mount)

![Union Mounting][unionmounting]

### Tooling

Anyone can use basic Linux features to implement containers, with no additional software. For example, [bocker](https://github.com/p8952/bocker) is a basic 100% Bash Docker clone, implemented with built-in Linux features and a couple of extra utilities like curl.

What container engines and software packages add on top of the basic OS features are helper functions, repeatability, automation, and infrastructure. There are many different containerization interfaces, including, but not limited to:

* Canonical's **LXC** and its successor **LXD** (from the makers of Ubuntu)
* **Docker**, an open-source product sponsored by Docker, Inc.
* **rkt** (rocket), a community attempt to build a more open, community-driven container engine

For the remainder of this presentation, we will work exclusively with the most popular container platform, Docker, but it's important to remember that Docker is only one take on containerization tooling, and that other valid (and sometimes interoperable, since they all use the same OS features under the hood) alternatives exist.

## Docker fundamentals

Docker provides a lot of higher-level concepts, APIs, apps, and infrastructure on top of the basic virtualization capabilities of Linux.

### Basic Docker elements

#### Image

An image is a read-only template with instructions for creating a Docker container. Often, an image is based on another image, with some additional customization. For example, you may build an image which is based on the ubuntu image, but installs the Apache web server and your application, as well as the configuration details needed to make your application run. [docker](https://docs.docker.com/engine/docker-overview/#docker-objects)

It's important to realize that docker images are _built_ from a set of instructions called a _Dockerfile_. The image is not checked into source control; the _Dockerfile_ is. Anyone can clone the repo containing the Dockerfile and build their own copy of the image for themselves, by replaying the original instructions in the Dockerfile.

##### Sample Dockerfile
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

#### Container

A container is a runnable instance of an image. You can create, start, stop, move, or delete a container using the Docker API or CLI. You can connect a container to one or more networks, attach storage to it, or even create a new image based on its current state. [docker](https://docs.docker.com/engine/docker-overview/#docker-objects)

### Docker Engine

A client/server application, and what is generically meant when someone says the word "docker" with no further elaboration. It has these components:

* A daemon process (`dockerd`) that manages running Docker containers as services, and provides many utility functions. This is where the magic happens.
* A REST API that clients can use to manipulate `dockerd`
* A client-side CLI (`docker`) that, through the `dockerd` REST API, is used to work with containers, images, other Docker artifacts.

![Docker Architecture][dockerarchitecture]

### Other Docker infrastructure

#### Registry

The Docker Registry is a stateless, highly scalable server side application that stores and lets you distribute prebuilt Docker images. The Registry is open-source, under the permissive Apache license. [docker](https://docs.docker.com/registry/)

The most famous public Docker registry is the [Docker Hub](https://hub.docker.com/). This is similar to a public package implementation, like PyPI, npm, or NuGet, and is generally treated the same way by users.

Anyone can run a Docker registry, and many organizations choose to implement their own private registry. This is analogous to an organization running their own package library, such as NuGet, and is done for the same reasons.

### Docker on Windows

Docker, Inc. also maintains a Windows native version of Docker.

#### Linux Containers

Docker for Windows can use Hyper-V to run a fully virtualized Linux environment, which, in turn, hosts containers. All normal docker commands are implemented in the Windows executable. These commands are delegated to the instance of Docker that runs within the Linux VM. This results in a fairly seamless experience for Windows developers, with some caveats:

* Enabling Hyper-V, by its very nature, disables all other VM products. You can't use VirtualBox, VMWare, or anything else while Hyper-V is turned on, and therefore while Docker for Windows is enabled. You can set up a boot menu to toggle Hyper-V on and off with a reboot.

#### Windows containers

Windows 10 (Professional or Enterprise) and Windows Server 2016 offer _native support_ for Windows containers. Windows containers are a Windows-native container implementation that I understand much less well than Linux containers and therefore cannot offer very much technical information on at this time. I understand that two different implementations are available:

* Windows Server Containers, which are similar in nature to Linux containers, using process and namespace isolation mechanisms similar to Linux's cgroups and namespaces. However, this mechanism is not considered secure enough to run untrusted code, so Microsoft also offers
* Hyper-V isolation, which is basically just using a super-optimized Hyper-V VM to provide container isolation

Windows containers can switch between these isolation mechanisms without additional configuration, making this a run-time decision, not a design-time one.

Docker for Windows can run against and automate Windows containers the same way that it does for Linux containers. On a Windows system, Docker for Windows can be switched to operate against Windows containers or Linux containers, but cannot operate against both at the same time.

## Demos

* Development workflow
  * Build and run some code in a container
  * Run same code in a different container on second host
* Appliance hosting
  * Build and run some other code in a second container on a third host that depends on the code from the first example, and runs it as an appliance
* AWS: ECS
  * Push both containers to ECS and run them there

## Uses

Docker has many uses, including, but not limited to:

* **Environment consistency**: Run the same code in an repeatable, identical environment, from dev straight up to prod
* **Infrastructure as code**: All configuration and setup for a container are repeatable and stored in a repo
* **Simplified dev environment**: run server dependencies as appliances instead of as native installations in the host environment
* **Ease dependency hell**: Different containers can have their own library versions on the same host environment

[vmsvscontainers]:https://images.idgesg.net/images/article/2017/06/virtualmachines-vs-containers-100727624-large.jpg
"From the web"

[unionmounting]: http://cecs.wright.edu/~pmateti/Courses/3900/Lectures/Internals/Figures/mount-bind.png
"From the web"

[dockerarchitecture]:
https://docs.docker.com/engine/images/architecture.svg
"docker.com"