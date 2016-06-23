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

contact: nagin@il.ibm.com


3) Specify FIWARE Docker Container Service URL as the *DOCKER_HOST* endpoint

In order to prepare your docker client to interact with the FIWARE Docker Container Service you need to export the service's URL to the DOCKER_HOST environment variable or reference the URL in each docker command's -H <services URL>

    >export DOCKER_HOST=130.206.119.3:2376

or

    >docker -H 130.206.119.3:2376

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

There are many ways to get your keystone token id and tenant id.  For instance you could use curl.  But we have provided and a script called [set-docker-config.sh](./set-docker-config.md) that makes creating your config.json file easy.
  
Create default docker configuration file, .docker/config.json, for default:

    >set_docker_conf.sh -t <fiware-tenant-name> -u <fiware-user-name> -p <user-password>
Run docker commands:

    >docker <cmd> 

Keystone token's expire after appproximately one day so you will need to update the configuration file daily.

For advanced users of set-docker-conf.sh script see [set-docker-config](./set-docker-config.md).

## Authorization

Communication with the service is through REST.  Tenant authorization is accomplished by specifying a valid FIWARE keystone token and tenant id in http  headers "X-Auth-Token" and "X-Auth-TenantId".  The token owner must be authorized to use the service.  A request will be rejected if the token has expired or the token is not authorized to access the specified tenant.

## Managing containers

Docker requests to manage your containers are supported, but they are limited to containers that belong to the specified tenant.  For instance ps will only list continainers associated with the specificed tenant.

There are some restictions on docker requests to run and create containers. In brief the restrictions are related to the service isolating tenants from each other and from the docker host.  Thus in addition to limiting a tenant to only managing their own containers, tenants are prevented from disproportionately utilize resources. 

## Networking containers

The port bindings to the host external ports are restricted to those that the docker auto assigns in the ephermal port range which typically ranges from 32768 to 61000.  Thus the -P or --publish-all flags are supported but specifying a value in the host port with the -p or --publish flag are rejected.

Currrently, private network creation is not supported but we intent provide support in the future.

## Managing container data

The local host should not be referenced in your docker commands.  Commands that reference the local host will be rejected.  Thus a command that references a docker host directory in the -v or --volume flags is rejected.

## Managing docker images

Currently the service does note not allow you build or manage images.  However you can pull images from [Docker Hub](https://docs.docker.com/docker-hub).

Note: we do plan to support managing images in private repositories in the future.


##Docker CLI
Once you prepare your docker client as described in [Quick Start](./user-guide.md##Quick Start) you can use
the [Docker CLI](https://docs.docker.com/engine/reference/commandline/cli/).
All the commands to manage your containers are supported. But they are limited to containers that belonging to the tenant specified in your config.json file.  So ps will only list containers belonging to the specified tenant. Likewise, there are some restictions on docker run and create.

See [Docker CLI Support](./docker-cli.md).


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
