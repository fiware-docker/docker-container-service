# FIWARE Docker Container Service (FDCS)


## Goal
The FIWARE Docker Container Service (FDCS) exposes the docker API so that FIWARE tenants can use their locally docker client to remotely manage their docker resources, i.e. containers, volumes, networks, etc.  However tenants are insulated from each other so they can only manage docker resources, that they have created. In addition to exposing the regular Docker capabilities FDCS provides authorization and authentication capabilities which allows it to support tenant isolation between different FIWARE accounts. The multi-tenant Docker support is integrated with FIWARE Keystone service and its existing accounts definitions. The service also provides support for name scoping between tenants such that tenants can use the same docker resource names without interfering with each other. 

FDCS is deployed on the FIWARE Lab at docker.lab.fiware.org:2376 so users can get its benefits by simply applying for admission to the service. Alternatively, FDCS can be installed on-premises so a site can deploy docker containers on their private FDCS cluster. 

The main capabilities provided by FDCS are:</p>

* Multi-tenant isolation 
* Name scoping 
* Manage life cycle Docker Hosts
* Manage network and storage attached Docker Hosts
* Resource monitoring of Docker Hosts
* Resiliency of the persistent data associated with Docker Hosts
* Manage resource allocation
* Secure access to the Docker Hosts
* Secure access to the Docker Containers 


## Rational
The service will simplify the usage of docker to develop FIWARE applications since tenant administrators will be relieved of the task of managing their docker hosts; the docker hosts will be managed by the FIWARE Lab infrastructure providers.  Further more since the docker hosts and related resources will be shared by multiple tenants this will allow the FIWARE lab to achieve greater density of containers per host.

#Description
Docker Swarm is native clustering for Docker. It allows access to a pool of Docker hosts using the full suite of Docker tools. Because Docker Swarm serves the standard Docker API, any tool that already communicates with a Docker daemon can use Swarm to transparently scale to multiple hosts.  The service enhances Swarm to support multi-tenant isolation and name scoping.

Docker client interacts with FDCS through the [Docker Remote API](https://docs.docker.com/engine/reference/api/docker_remote_api). The Docker API is a RESTful interface and all interactions are via well-known HTTP methods such as GET, PUT and DELETE. 
The only difference is that two additional headers are included in the docker REST request, namely 
     "X-Auth-Token": keystone token id and "X-Auth-TenantId": keystone tenant id

When the user's docker client is the docker cli, they should update the docker config.json to contain the headers:


     { "HttpHeaders":
       { "X-Auth-Token": <keystone token id>,
         "X-Auth-TenantId": <keystone tenant id>
       }
     }


When the Service receives a docker request it will contact FIWARE's Keystone Identity Manager to authorize the tenant's request based on the headers' token and tenant id.

The figure below illustrates this process.  

In step 1 the user sends an ''Keystone Authenticate'' request to Keystone describing a FIWARE user which is a member of a tenant.

In step 2,3 Keystone generates a token and returns it to the user. The user then updates their docker config.json file with the Keystone token and tenant id.
The use then invokes a docker cli command which transparently sends docker REST requests (including the token and tenant id) to the Service.

In step 6 the Service's multi-tenant swarm sends a ''Keystone list tenants'' request to Keystone which returns a list of tenants to which the token is authorized to access. 

In step 8 the Service's multi-tenant swarm checks whether the user's tenant id is in tenant list.  Once the the tenant has been authorized swarm ensures that the tenants are isolated from each other by ensuring that each tenant can only manage and link to their own containers, volumes, and networks.

![](figs/multi-tenant_Swarm_with_Keystone_Authentication.png?raw=true)

[FDCS User Guide](https://github.com/fiware-docker/docker-container-service/blob/master/docs/userguide/user-guide.md)  describes in more detail how the docker client may interact with the service.



## Basic Design Principles

Docker implements a high-level API to provide lightweight containers that run processes in isolation. FDCS exposes the Docker API and thus allows any FIWARE Tenant with a docker client to remotely create and manage containers.

In the illustration below we show a local Docker client remotely managing its Docker containers from a work station using the service. Most of the communication between the client and the service is through the Docker REST API. But the client must communicate with FIWARE’s Openstack Keystone Identity Management to get a valid token to interact with the service.
Once a valid token is obtained the client can use the docker cli or docker compose to create and deploy complex Docker services.

![](figs/FDCS_Overview.png?raw=true)


In the illustration below we show a FIWARE Docker Container Service's cluster of docker hosts. Three FIWARE tenants are sharing the cluster's resources, but the service ensures that tenants are isolated from each other.

![](figs/FDCS_Tenant_Isolation.png?raw=true)


For details about how to use FDCS see the [FDCS User Guide](userguide/user-guide.md).

For details about how to administer and install an on-premise FDCS see the [FDCS Admin and Installation Guide](adminguide/admin-guide.md).

