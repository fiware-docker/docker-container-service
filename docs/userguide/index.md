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
1) *Apply to get a FIWARE account*

You can get a FIWARE Account [here](https://account.lab.fiware.org/).

2) *Apply to get a FIWARE Docker Container Service Account*

**TO DO.  We need a FIWARE page for doing this!** 


3) Specify FIWARE Docker Container Service URL as  *DOCKER_HOST* endpoint
In order to prepare your docker client to interact with the FIWARE Docker Container Service you need export the service's URL to the DOCKER_HOST environment variable or reference the URL in each commands -H <services URL>

    >export DOCKER_HOST=130.206.126.17:2376

or

    >docker -H 130.206.126.17:2376

4) Set up *Docker configuration file*

The config.json file describes additional headers to include in the docker REST commands sent to the docker container servicer.  The the headers are X-Auth-Token and X-Auth-TenantId. 

The *config.json* takes the following form:

    { "HttpHeaders":
      {
        "X-Auth-Token": <keystone token id>,      
        "X-Auth-TenantId": <keystone tenant id>    
      }
    }


The default location of the configuration file is *$HOME/.docker*.  But you can use the docker --config flag to indicate another directory. 

There are many ways to get your keystone token id and tenant id.  For instance you could use curl.  But we have provided and utility called [set-docker-config](./set-docker-config.md) that makes creating your config.json file easy.
  
Create default docker configuration file, .docker/config.json, for default:

    >set_docker_conf.sh -t <fiware-tenant-name> -u <fiware-user-name> -p <user-password>
Run docker commands:

    >docker <cmd> 

Keystone token's expire after appproximately one day so you will need to update the configuration file daily.

For advanced users of set-docker-conf.sh script see [set-docker-config](./set-docker-config.md).
-

##Docker Commandline
##Docker API
## Docker Compose
Currently docker compose is not supported because it does not accept headers in the docker configuration file.

##Docker Image Repository Handling
Currently the service only supports pulls of public images from the docker hub.  Private repositories are not supported.

==Docker Hub==

Docker Hub is the central hub for Docker. It hosts public Docker images
and provides services to help you build and manage your Docker
environment. To learn more:

Go to [Using Docker Hub](https://docs.docker.com/docker-hub).

## Dockerizing applications: A "Hello world"

*How do I run applications inside containers?*

Docker offers a *container-based* virtualization platform to power your
applications. To learn how to Dockerize applications and run them:

Go to [Dockerizing Applications](dockerizing.md).


## Working with containers

*How do I manage my containers?*

Once you get a grip on running your applications in Docker containers
we're going to show you how to manage those containers. To find out
about how to inspect, monitor and manage containers:

Go to [Working With Containers](usingdocker.md).

## Working with Docker images

*How can I access, share and build my own images?*

Once you've learnt how to use Docker it's time to take the next step and
learn how to build your own application images with Docker.

Go to [Working with Docker Images](dockerimages.md).

## Networking containers

## Managing data in your containers

The local host should not be referenced in your docker commands.  Commands that reference the local host will be rejected

Go to [Managing Data in Containers](dockervolumes.md).


 




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
