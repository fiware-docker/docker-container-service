<!--[metadata]>
+++
title = "FIWARE Docker Container Service user guide"
description = "FIWARE Docker Container Service programmer and user guide user guide home page"
keywords = ["docker, introduction, documentation, about, technology, docker.io, user, guide, user's, manual, platform, framework, virtualization, home,  intro"]
[menu.main]
parent = "mn_fun_docker"
+++
<![end-metadata]-->

#FIWARE Docker Container Service user guide
##Introduction
The FIWARE Docker Container Service exposes the docker API so that FIWARE users can leverage their local docker clients to remotely manage their docker containers on the FIWARE Lab. This document describes how to use the service.

##Quick Start
1) *Apply to get a FIWARE Lab account*

You can get a FIWARE Lab account [here](https://account.lab.fiware.org/).

2) *Apply to get a FIWARE Docker Container Service Account*

contact: [nagin@il.ibm.com](mailto:nagin@il.ibm.com?subject=FDCS%20Application)


3) *Set up your local docker client environment*

* Specify FIWARE Docker Container Service URL, i.e tcp://docker.lab.fiware.org:2376, as the *DOCKER_HOST* endpoint

In order to prepare your docker client to interact with the FIWARE Docker Container Service you need to export the service's URL to the DOCKER_HOST environment variable or reference the URL in each docker command's -H <services URL>

    >export DOCKER_HOST=tcp://docker.lab.fiware.org:2376

or

    >docker -H tcp://docker.lab.fiware.org:2376

* Set up *Docker configuration file*

The config.json file describes additional headers to include in the docker REST commands sent to the docker container servicer.  FDCS requires headers X-Auth-Token and X-Auth-TenantId. X-Auth-Token references the user's Keystone token. X-Auth-TenantId references user's Keystone tenant id.

The *config.json* takes the following form:

    { "HttpHeaders":
      {
        "X-Auth-Token": <keystone token>,      
        "X-Auth-TenantId": <keystone tenant id>    
      }
    }


The default location of the configuration file is *$HOME/.docker*.  But you can use the docker --config flag to indicate another directory. 

There are many ways to get your keystone token id and tenant id.  For instance you could use curl.  But we have provided and a script called [set-docker-config.bash](https://github.com/fiware-docker/docker-container-service/tree/master/docs/userguide/set-docker-config.md) that makes creating your config.json file easy.
  
This is an example of using the script to create a docker configuration file at .docker/config.json: 

    >set_docker_conf.bash -t <fiware-tenant-name> -u <fiware-user-name> -p <user-password>

Keystone tokens expire after appproximately one day so you will need to update the configuration file daily.

For more information about the script see the [set_docker_config readme](https://github.com/fiware-docker/docker-container-service/tree/master/docs/userguide/set-docker-config.md).


4) *Run docker commands:*

    > docker run hello-world
    Hello from Docker.
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

     To try something more ambitious, you can run an Ubuntu container with:
      $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker Hub account:
     https://hub.docker.com

    For more examples and ideas, visit:
     https://docs.docker.com/userguide/
 
    > docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                             NAMES
	c1d22839920e        hello-world         "/hello"                 20 seconds ago      Exited (0) 17 seconds ago                                     docker-host-2
    > rm c1d22839920e


## Authentication and Authorization

Communication with the service is through REST.  Tenant authentication and authorization is accomplished by specifying a valid FIWARE keystone token and tenant id in http headers "X-Auth-Token" and "X-Auth-TenantId".  The token is authenticated by Keystone, its owner must be associated with the tenant id, and token owner must be authorized to use the service.  A request will be rejected if the above conditions are not met or the token has expired.

## Multi-Tenant Isolation

FDCS supports mulit-tenant isolation. FDCS allows many authorized FIWARE tenants to use the FDCS cluster, however the tenants are isolated from each other.  Tenants can create docker resources, i.e. containers, networks, volumes, etc., but only the tenant that creates a docker resource may manage the resource.  Tenants can not even see other tenants' resource let alone manage them.

## Multi-Tenant Name Scoping

FDCS supports multi-tenant name scoping.  This means tenants can assign the same name to their docker resources without interfering with each other.  For instants two tenants could create a container with the same name. They can also reference their resources by name and FDCS will ensure that the reference is to the resource owned by the tenant.

## Managing containers

All docker requests to manage containers are supported, but they are limited to containers that belong to the specified tenant.  For instance ps will only list containers created by the specified tenant.

There are some restrictions on docker requests to run and create containers. In brief the restrictions are related to the service isolating tenants from each other and from the docker host.  Thus in addition to limiting a tenant to only managing their own containers, tenants are prevented from disproportionately over utilizing resources. 

## Networking containers
### User Defined Networks
FDCS supports user defined networks and their management.  Currently it only supports bridge type networks.  In the future it will support overlay networks. Containers on the same network can securely communicate with each other, but those on different networks are isolated from each.  Each network has its own DNS which allows communication between containers on the same network using container names.

### linking containers
FDCS supports the docker link feature.  The docker link feature allows containers to discover each other and securely transfer information about one container to another container.  However, links are being deprecated by docker so it is preferred to leverage user defined networks when networking your containers.

### host port mapping
Host port mapping allows public access to containers.  Host ports may be bound to a container ports so that the container listens for incoming traffic on its port while the actual traffic is being directed at a host port. Port bindings are restricted to those host ports that the docker auto assigns in the ephermal port range which typically ranges from 32768 to 61000.  Thus the -P or --publish-all flags are supported but specifying a value in the host port with the -p or --publish flag are rejected.


## Managing container data
### Data Volume Containers
FDCS supports *Data Volume Containers* and their management.  If you have some persistent data that you want to share between containers, or want to use from non-persistent containers, you can create a named *Data Volume Container*. Containers that want to share the data can reference the *Data Volume Container* with the create volume --volumes-from flag. *Data Volume Container* data persists even when the containers that reference the data volume container are removed.

### User Defined Volumes
FDCS supports user defined volumes and their management.  Multiple containers can use the same volume in the same time period. This is useful if two containers need access to shared data. For example, if one container writes and the other reads the data. User defined volume data persist even when the containers that reference the volume are removed. Further since FDCS mounts user defined volumes on a NFS mount point the volumes are shared by all the host in the FDCS cluster. Thus, containers on different hosts can easily share data.  
### host volume mounts
The docker create container command may mount a host volume with it's -v or --volume flags.  However, FDCS does not allow this, since the user can not have direct access to FDCS's docker hosts. Thus a command that attempts to mount a host volume in the -v or --volume flags is rejected. 

Typically host volume mounts are used to configure a container from files in the host file system. We have seen that most container configurations can be resolved with environment variables and/or using commands, like wget, to fetch the configuration data from an internet repository.  Another solution is to create a custom image locally and push it to docker hub from where it can be pulled.   

## Managing docker images

Currently the service does not allow you to build or manage images.  However you can pull images from [Docker Hub](https://docs.docker.com/docker-hub).

Note: we do plan to support managing images in private repositories in the future.


##Docker CLI
Once you prepare your docker client as described in the user guide's Quick Start you can use the [Docker CLI](https://docs.docker.com/engine/reference/commandline/cli/). All the commands to manage your containers are supported. But they are limited to containers that belonging to the tenant specified in your config.json file.  So ps will only list containers belonging to the specified tenant. Likewise, there are some restictions on docker run and create.

See [Docker CLI Support](https://github.com/fiware-docker/docker-container-service/tree/master/docs/userguide/docker-cli.md).

##docker-compose
Once you prepare your docker client as described in the user guide's Quick Start you can use [Docker Compose](https://docs.docker.com/compose/). We support Docker-Compose 1.6.2 and above.

Note: docker-compose does not support the docker cli --config flag, so the ~/.docker/config.json must contain headers X-Auth-Token and X-Auth-TenantId. Likewise, docker-compose does not support the docker cli -H flag so the environment variable must be set to tcp://docker.lab.fiware.org:2376.

See [docker-compose Support](https://github.com/fiware-docker/docker-container-service/tree/master/docs/userguide/docker-compose.md).


## Getting Docker help

* [Docker homepage](https://www.docker.com/)
* [Docker Hub](https://hub.docker.com)
* [Docker blog](https://blog.docker.com/)
* [Docker documentation](https://docs.docker.com/)
* [Docker Getting Started Guide](https://docs.docker.com/mac/started/)
* [Docker code on GitHub](https://github.com/docker/docker)
* [Docker mailing
  list](https://groups.google.com/forum/#!forum/docker-user)
* Docker on IRC: irc.freenode.net and channel #docker
* [Docker on Twitter](https://twitter.com/docker)
* Get [Docker help](https://stackoverflow.com/search?q=docker) on
  StackOverflow
* [Docker.com](https://www.docker.com/)
