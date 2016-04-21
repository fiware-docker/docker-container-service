<!--[metadata]>
+++
title = "FIWARE Docker Container Service Installation and Administration Guide"
description = "FIWARE Docker Container Service Installation and Administration Guide home page"
keywords = ["docker, introduction, documentation, about, technology, docker.io, user, guide, user's, manual, platform, framework, virtualization, home,  intro"]
[menu.main]
parent = "mn_fun_docker"
+++
<![end-metadata]-->

#FIWARE Docker Container Service Installation and Administration Guide
##Deployment Steps
This section describes the procedure for manually deploying a Docker Container Service on OpenStack.  In brief, the Docker Container Service is a Multi-Tenant Swarm cluster.  We refer to the node where the Multi-Tenant Swarm manager is running as the Swarm Management Node and nodes where the docker engines are running as the Docker Nodes.  The following steps are required:
<ol>
<li> Create an SSH key pair that will be used to access the Swarm Management Node, the Docker Nodes in the Swarm cluster, and NFS Server

<li> Create a security group for the Swarm Management Node.  It containers rules for allowing public access to the Swarm Manager Port, SSH port, and Ping. For example:

|service       | IP Protocol | From Port | To Port | Source            |
|--------------|-------------|-----------|---------|-------------------|
| SSH          | TCP         | 22        | 22      | 0.0.0.0/0 (CIDR)  |
| Ping         | ICMP        | 0         | 0       | 0.0.0.0/0 (CIDR)  |
| Swarm Manager| TCP         | 2376      | 2376    | 0.0.0.0/0 (CIDR)  |

 <table style="width:100%">
  <tr>
    <td>service</td>
    <td>IP Protocol</td>
    <td>From Port</td>
    <td>To Port</td>
    <td>Source<td>
  </tr>
  <tr>
    <td>SSH</td>
    <td>TCP</td>
    <td>22</td>
    <td>22</td>
    <td> 0.0.0.0/0 (CIDR)<td>
  </tr>
    <tr>
    <td>Ping</td>
    <td>ICMP</td>
    <td>0</td>
    <td>0</td>
    <td> 0.0.0.0/0 (CIDR)<td>
  </tr>
    <tr>
    <td>Swarmp Manager</td>
    <td>TCP</td>
    <td>2376</td>
    <td>2376</td>
    <td> 0.0.0.0/0 (CIDR)<td>
  </tr>


</table> 

<li> Create a Swarm Management Node VM instance.  Associate it with its security group and key pair. Optionally install Docker on the Swarm Management Node to allow you to use the docker cli to send docker commands from the Swarm Management Node to the Docker Nodes for debug purpose.

<li> Create a security group for the Docker nodes.  It containers rules for allowing public access to the SSH port, Ping, and docker auto assigned ports.  The docker auto assigned ports are those ports that docker automatically assigns to containers as there external ports when they are not specifically designated in the docker command.  It also containers a rule for exclusive access to the Docker port from the Swarm Management Node.  Use the Swam Management Node’s public IP.  

service       | IP Protocol | From Port | To Port | Source 
------------- | ------------| --------- | ------- | ----------------
SSH           | TCP         | 22        | 22      | 0.0.0.0/0 (CIDR)
Ping          | ICMP        | 0         | 0       | 0.0.0.0/0 (CIDR)
Docker Engine | TCP         | 2375      | 2375    | <Swarm Manager Public IP/32 (CIDR)
Docker Containers auto assigned by docker engine   | TCP         | 32768     | 32768   | 0.0.0.0/0 (CIDR)
 
<li> Create docker nodes.  Associate them with their security group and ssh keypair. 
Install Docker on all the Docker Node instances (http://docs.docker.com/engine/installation/).  
Enable swap cgroup memory limit following those steps from docker  documentation https://docs.docker.com/engine/installation/ubuntulinux/

<li> Create a security group for the NFS server.  It contains rules for ssh access and for servicing the Docker Nodes

<li> Create a NFS Server.  Associate it with its security group and key pair.

<li> Start the docker engines daemon to listen on the port that was specified when you created the Docker Nodes security group in step <4>.  Optionally, you can allow it to also listen on a linux file socket to simplify debug. >sudo docker daemon -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 – icc=false

<li> Configuring Swarm:
   Environment Variables:
      -- SWARM_MULTI_TENANT: if set to false vanila swarm is run with out multi-tenancy or name scoping.
      -- SWARM_AUTH_BACKEND: if set to Keystone then Keystone is used to authenticate and authorization docker requests based on the Authorization Token and Tenant ID in their request header. 
      -- SWARM_MEMBERS_TENANT_ID: contains the id of the tenant whose members are eligible to use the service. If not set then any valid token tenant id may use the service. Only only valid when SWARM_AUTH_BACKEND is set to Keystone.
      -- SWARM_ADMIN_TENANT_ID: contains the id of the tenant that may run docker commands as admin. 
      -- SWARM_FLAVORS_ENFORCED
      -- SWARM_FLAVORS_FILE
      -- SWARM_CONFIG

<li> Start  Multi-Tenant  Swarm Manager daemon (without TLS) on the Swarm Management Node.  If token discovery is to be used then add the discovery flag, otherwise use the file flag to point to a file with a list of all the Docker Node public ips and docker ports. 

<li> Test the cluster’s remote connectivity by pinging and sshing to all the instances (including the Swarm Management Node). 

<li> Test whether the Multi-Tenant Swarm Cluster works as expected by using docker commands on your local docker client.  The docker –H flag specifies the Swarm Manager Node and swarm port.  The docker –config specifies the directory where a config.json file is prepared with a valid token and a valid tenantid. 
</ol>  

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
