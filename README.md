# FIWARE Docker Container Service


# Goal
The FIWARE Docker Container Service (FDCS) exposes the docker API so so that FIWARE tenants can use their docker client remotely to manage their containers.  However tenants are insulated from each other so they can only manage containers that they have created. In addition to exposing the regular Docker capabilities FDCS provides authorization and authentication capabilities which allows it to support tenant isolation between different FIWARE accounts. The multi-tenant Docker support is integrated with FIWARE Keystone service and its existing accounts definitions. The service also provides support for name scoping between tenants such that tenants can use the same docker resource names without interfering with each other. 

FDCS is deployed as on the FIWARE Lab so users can get its benefits by simply applying for admission to the service. Alternatively, FDCS can be installed on-premises so a site can deploy docker containers on their private FDCS cluster. 

<p>The main capabilities provided by FDCS are:</p>
<ul>
 <li>Multi-tenancy (support isolation between Contains of different accounts) 
 <li> Name scoping (the same docker resource name can be used between tenants without interfering with each other )
<li>Manage life cycle Docker Hosts
<li>Manage network and storage attached Docker Hosts
<li>Resource monitoring of Docker Hosts
<li>Resiliency of the persistent data associated with Docker Hosts
<li>Manage resource allocation
<li>Secure access to the Docker Hosts
<li>Secure access to the Docker Containers 
</ul>

#Rational
The service will simplify the usage of docker to develop FIWARE applications since tenant administrators will be relieved of the task of managing their docker hosts; the docker hosts will be managed by the FIWARE Lab infrastructure providers.  Further more since the docker hosts and related resources will be shared by multiple tenants this will allow the FIWARE lab to achieve greater density of containers per host.

#Description
Docker Swarm is native clustering for Docker. It allows access to a pool of Docker hosts using the full suite of Docker tools. Because Docker Swarm serves the standard Docker API, any tool that already communicates with a Docker daemon can use Swarm to transparently scale to multiple hosts.  The service enhances Swarm to support multi-tenancy.

The interaction with the Docker GE is done through the Docker Rest API.
Docker API is a RESTful interface and all interactions are via well-known HTTP methods such as GET, PUT and DELETE. For complete details on how a client interacts with the Docker API, please refer to [https://docs.docker.com/engine/reference/api/docker_remote_api Docker Remote API] 

The [https://docs.docker.com/reference/api/docker_remote_api/ Docker Remote API] is used by docker clients to interact with the FIWARE Docker Container Service.
The only difference is that two additional headers are included in the docker REST request, namely 
     '''"X-Auth-Token": <keystone token id>, "X-Auth-TenantId": <keystone tenant id>'''

When the user's docker client is the docker cli, they should update the docker config.json to contain the headers:


     { "HttpHeaders":
       { "X-Auth-Token": <keystone token id>,
         "X-Auth-TenantId": <keystone tenant id>
       }
     }


When the Service receives a docker request it will contact FIWARE's Keystone Identity Manager to authorize the tenant's request based on the headers' token and tenant id.
The figure below illustrates this process.  In step 1 the user sends an ''Keystone Authenticate'' request to Keystone describing a FIWARE user which is a member of a tenant.
In step 2,3 Keystone generates a token and returns it to the user. The user then updates their docker config.json file with the Keystone token and tenant id.
The use then invokes a docker cli command which transparently sends docker REST requests (including the token and tenant id) to the Service.
In step 6 the Service's multi-tenant swarm sends a ''Keystone list tenants'' request to Keystone which returns a list of tenants to which the token is authorized to access. 
In step 8 the Service's multi-tenant swarm checks whether the user's tenant id is in tenant list.  Once the the tenant has been authorized swarm ensures
that the tenants are isolated from each other by ensuring that each tenant can only manage and link to their own containers, volumes, and networks.

<img src="https://github.com/fiware-docker/docker-container-service/blob/master/figs/multi-tenant_Swarm_with_Keystone_Authentication.png" alt="text" />

![](https://github.com/fiware-docker/docker-container-service/blob/master/figs/multi-tenant_Swarm_with_Keystone_Authentication.png?raw=true)

[FDCS User Guide](https://github.com/fiware-docker/docker-container-service/blob/master/docs/userguide/user-guide.md)  describes in more detail how the docker client may interact with the service.



= Basic Design Principles =


Docker implements a high-level API to provide lightweight containers that run processes in isolation.
It's basic design principles are described succinctly in [https://en.wikipedia.org/wiki/Docker_%28software%29 Docker's wikipedia page].

FDCS exposes the Docker API and thus allows any FIWARE Tenant with a docker client to remotely create and manage containers.
In the illustration below we show a local Docker client remotely managing its Docker containers from a work station using the service.
Most of the communication between the client and the service is through the Docker REST API.
But the client must communicate with FIWARE’s Openstack Keystone Identity Management to get a valid token to interact with the service.
Once a valid token is obtained the client can use the docker cli or docker compose to create and deploy complex Docker services.

[[File:FIWARE Docker Container Service with Multi-Tenant Swarm.png|center]]


In the illustration below we show a FIWARE Docker Container Service's cluster of docker hosts.
Three FIWARE tenants are sharing the cluster's resources, but the service ensures that tenants are isolated from each other.

[[File:Tenant_Isolation.png|center]]




