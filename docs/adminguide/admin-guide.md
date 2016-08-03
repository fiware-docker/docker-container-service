<!--[metadata]>
+++
title = "FIWARE Docker Container Service (FDCS) Installation and Administration Guide"
description = "FIWARE Docker Container Service Installation and Administration Guide home page"
keywords = ["docker, introduction, documentation, about, technology, docker.io, user, guide, user's, manual, platform, framework, virtualization, home,  intro"]
[menu.main]
parent = "mn_fun_docker"
+++
<![end-metadata]-->

#FIWARE Docker Container Service Installation and Administration Guide
##Deployment Steps
This section describes the procedure for manually deploying a FIWARE Docker Container Service (FDCS) on OpenStack.  In brief, the FDCS is a Multi-Tenant Swarm cluster.  We refer to the node where the Multi-Tenant Swarm manager is running as the Swarm Management Node and nodes where the docker engines are running as the Docker Nodes.  The following steps are required:

<ol>
<li>Create an SSH key pair that will be used to access the Swarm Management Node, the Docker Nodes in the Swarm cluster, and NFS Server

<li>Create a security group for the Swarm Management Node.  It containers rules for allowing public access to the Swarm Manager Port, SSH port, and Ping. For example:

 <table style="width:100%">
  <tr>
    <th><b>service</b></th>
    <th><b>IP Protocol</b></th>
    <th><b>From Port</b></th>
    <th><b>To Port</b></th>
    <th><b>Source</b><th>
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


<li>Create a Swarm Management Node VM instance.  Associate it with its security group and key pair. Install Docker on the Swarm Management Node which will be used to launch the docker image of the Multi-Tenant Swarm.

<li>Create a security group for the Docker nodes.  It containers rules for allowing public access to the SSH port, Ping, and docker auto assigned ports.  The docker auto assigned ports are those ports that docker automatically assigns to containers as there external ports when they are not specifically designated in the docker command.  It also containers a rule for exclusive access to the Docker port from the Swarm Management Node.  Use the Swam Management Node’s public IP.  

 <table style="width:100%">
  <tr>
    <th><b>service</b></th>
    <th><b>IP Protocol</b></th>
    <th><b>From Port</b></th>
    <th><b>To Port</b></th>
    <th><b>Source</b></th>
  </tr>
  <tr>
    <td>SSH</td>
    <td>TCP</td>
    <td>22</td>
    <td>22</td>
    <td> 0.0.0.0/0 (CIDR)</td>
  </tr>
  <tr>
    <td>Ping</td>
    <td>ICMP</td>
    <td>0</td>
    <td>0</td>
    <td> 0.0.0.0/0 (CIDR)</td>
  </tr>
  <tr>
    <td>Docker Engine</td>
    <td>TCP</td>
    <td>2375</td>
    <td>2375</td>
    <td> Swarm Manager Public IP/32 (CIDR)</td>
  </tr>
    <tr>
    <td>Docker Containers auto assigned by docker engine</td>
    <td>TCP</td>
    <td>32768</td>
    <td>61000</td>
    <td> 0.0.0.0/0 (CIDR) </td>
  </tr>
  </table>

<li> Create docker nodes.  Associate them with their security group and ssh keypair. 
[Install Docker on all the Docker Node instances](https://docs.docker.com/v1.11/).  
Enable swap cgroup memory limit following those steps from [docker documentation]( https://docs.docker.com/v1.11/engine/installation/linux/ubuntulinux/)

<li> Create a security group for the NFS server.  It contains rules for ssh access and for servicing the Docker Nodes.

 <table style="width:100%">
  <tr>
    <th><b>service</b></th>
    <th><b>IP Protocol</b></th>
    <th><b>From Port</b></th>
    <th><b>To Port</b></th>
    <th><b>Source</b></th>
  </tr>
  <tr>
    <td>SSH</td>
    <td>TCP</td>
    <td>22</td>
    <td>22</td>
    <td> 0.0.0.0/0 (CIDR)</td>
  </tr>
  <tr>
    <td>Ping</td>
    <td>ICMP</td>
    <td>0</td>
    <td>0</td>
    <td> 0.0.0.0/0 (CIDR)</td>
  </tr>
  <tr>
    <td>NFS Server</td>
    <td>TCP</td>
    <td>2049</td>
    <td>2049</td>
    <td> Docker Nodes  IP/32 (CIDR)</td>
  </tr>
  <tr>
    <td>NFS Server</td>
    <td>UDP</td>
    <td>2049</td>
    <td>2049</td>
    <td> Docker Nodes  IP/32 (CIDR)</td>
  </tr>
</table>


<li> Create a NFS Server.  Associate it with its security group and key pair.
<li> Install, configure and start the NFS Server:

    <p><b> 
     >apt-get install nfs-kernel-server
    </b></p>
    
     Create a directory that will be used to mount the docker volumes on the docker nodes:
    <p>
    <b>>sudo mkdir /docker_volumes</b>
    </p>
     In <b>/etc/hosts/</b> allow access to the docker volume directory to the docker nodes.  For instance:
     <p>
    <b>/docker_volumes <docker node ip>/24(rw,sync,no_subtree_check,no_root_squash)</b>.
    </p>
     Start the nfs server:
     <p>
    <b>>sudo service nfs-kernel-server restart</b>
    </p>

<li> On the docker nodes install the NFS client and configure it to use the nfs mount point:
    <p> 
    <b>>sudo apt-get install common-nfs</b>
    </p>
     Mount the docker volumes directory to be backed by nfs:
     <p>
    <b>>sudo mount -t nfs -o proto=tcp,port=2049 <nfs server ip>:/docker_volumes /var/lib/docker/volumes</b>
    </p>
     

<li> The engine daemon will be running as a service on each node
Configure the engine to listen on the port that was specified when you created the Docker Nodes security group above.  You should also allow it to listen on a linux file socket to simplify debug.  

     Update <b>/etc/default/docker</b> with 
     <p>
    <b>DOCKER_OPTS="-H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 --icc=false"</b>. 
    </p>
     And then start docker as a service:
     <p>
    <b>>sudo service docker restart</b>
    </p>

<li> Configuring Swarm:
   FDCS can be configured by setting its environment variables. FDCS environment variables are briefly described below:
   <ul>
      <li> <b>SWARM_ADMIN_TENANT_ID</b>: contains the id of the tenant that may run docker commands as admin. Admin is authorized to manage docker resources, including tenant containers, volumes, network, etc., and issue all docker requests without any filtering.
       
      <li> <b>SWARM_APIFILTER_FILE</b>: may point to a json file that describes the docker commands an installation of the service wants to filter out. If no file is pointed to it defaults to apifilter.json in the directory where swarm is started. 
      
Currently there is only support for a "disableapi" array which
contains a comma separated list of commands to disable.
This is an example how an installation could disable network support:       
<pre><code>
{
  "disableapi": [ networkslist, networkinspect, networkconnect, networkdisconnect, networkcreate, networkdelete ]
}
</pre></code>      
      
      <li> <b>SWARM_AUTH_BACKEND:</b> if set to "Keystone" then Keystone is used to authenticate Tenants that docker requests based on the Authorization Token and Tenant ID in their request header.
      
      <li> <b>SWARM_ENFORCE_QUOTA:</b> if set to "true" then the Multi-Tenant Swarm Quota feature is enabled otherwise it is disabled.  See <b>SWARM_QUOTA_FILE</b> for how to specify quotas.
      
      <li> <b>SWARM_FLAVORS_ENFORCED:</b> if set to "true" then the Multi-Tenant Swarm Flavors feature is enabled otherwise it is disabled. The  flavors specification is embodied in a json file which contains a map describing the valid resource combinations that can appear in create container requests.  Currently, Memory is the only resource that can be specified as a flavor. The Memory resource should be specified as a whole number which represents  megabytes of memory.   See <b>SWARM_FLAVORS_FILE</b> for how to specify flavors.
      
      <li> <b>SWARM_FLAVORS_FILE:</b> if SWARM_FLAVORS_ENFORCED is set to "true" then SWARM_FLAVORS_FILE points to a file with the flavors specification.  If there is no file pointed to then it defaults to flavors.json in the directory where swarm is started. 
      
The specification must contain a "default" flavor.  When the create container parameters do not match any of the specified flavors, the default flavor is applied to the create container replacing its original parameters. This is an example of the json flavors specification that is shipped with Multi-Tenant Swarm:       
<pre><code>       
{
  "default":{
   "Memory": 64
  },
  "medium":{
   "Memory": 128
  },
  "large":{
   "Memory": 256
  }

}
</code></pre>
In the above flavors specification example there are three flavors default, medium, and large.  <i>default</i> describes 64 megabytes of memory. <i>medium</i> describes 128 megabytes of memory. <i>large</i> describes 256 megabytes of memory. This means that a create container is limited to specifying its memory as 64MB, 128MB, or 256MB. If none is specified then the system will apply the default, i.e. 64MB. 

      <li> <b>SWARM_KEYSTONE_URL:</b> if SWARM_AUTH_BACKEND is set to "Keystone" then SWARM_KEYSTONE_URL must specify Keystone's URL, e.g. http://cloud.lab.fi-ware.org:4730/v2.0/.  
      
      <li> <b>SWARM_NETWORK_AUTHORIZATION:</b> if set to "false" then the Multi-Tenant Swarm Network Authorization feature is disabled otherwise it is enable.
      
      <li> <b>SWARM_MEMBERS_TENANT_ID</b>: contains the tenant id whose members are eligible to use the service. If not set then any valid token tenant id may use the service. SWARM_MEMBERS_TENANT_ID is only valid when SWARM_AUTH_BACKEND is set to Keystone.
      
      <li> <b>SWARM_MULTI_TENANT:</b> if set to "false" then the Multi-Tenant Swarm is disabled otherwise Multi-Tenant Swarm is enabled. When Multi-Tenant Swarm is disabled the result is that the service is launched as vanilla Swarm. Generally disabling Multi-Tenant Swarm is used for debugging purposes to discover if a bug is related to the swarm docker configuration or to a Multi-Tenant Swarm feature.
      
        <li> <b>SWARM_QUOTA_FILE:</b> if SWARM_ENFORCE_QUOTA is set to "true" then SWARM_QUOTA_FILE may specify the quota specification.  If there is no file pointed to it defaults to quota.json in the directory where swarm is started.
Currently quota support is limited to tenant memory consumption and it is the same for all tenants.  This is an example a json quota specification:
<pre><code>
{
   "Memory": 300
}
</code></pre>

      
    </ul>

<li> Start Multi-Tenant Swarm Manager daemon (without TLS) on the Swarm Management Node.  The Multi-Tenant Swarm docker image resides in the FIWARE Docker Hub repository at <b>fiware/swarm_multi_tenant</b>(https://hub.docker.com/r/fiware/swarm_multi_tenant/) 
If token discovery is to be used then add the discovery flag, otherwise use the file flag to point to a file with a list of all the Docker Node public ips and docker ports.  For instance:
   <pre>
   <code>
   >docker run -t -p 2376:2375 -v /tmp/cluster.ipstmp/cluster.ips -e SWARM_AUTH_BACKEND=Keystone -e SWARM_KEYSTONE_URL=http://cloud.lab.fi-ware.org:4730/v2.0/ -t fiware/swarm_multi_tenant:v0 --debug manage  file:///tmp/cluster.ips
   </code>
   </pre>

<li> Test the cluster’s remote connectivity by pinging and sshing to all the instances (including the Swarm Management Node). 

<li> Test whether the Multi-Tenant Swarm Cluster works as expected by using docker commands on your local docker client.  The docker –H flag specifies the Swarm Manager Node and swarm port.  The docker –config specifies the directory where a config.json file is prepared with a valid token and a valid tenantid.  For instance:

    <p><b>    
    >docker –H tcp://<Swam Manager Node IP>:2376  --config $HOME/dir docker command
    </b></b>

See the FIWARE Docker Container Service Users Guide for more details on how to use the  service.

</ol>  

## Getting Docker help

* [FIWARE Docker Container Service Users Guide](https://github.com/fiware-docker/docker-container-service/blob/master/docs/userguide/user-guide.md)
* [Docker homepage](https://www.docker.com/)
* [Docker Hub](https://hub.docker.com)
* [Docker blog](https://blog.docker.com/)
* [Docker documentation](https://docs.docker.com/)
* [Docker Getting Started Guide](https://docs.docker.com/v1.11/engine/quickstart/)
* [Docker code on GitHub](https://github.com/docker/docker)
* [Docker mailing
  list](https://groups.google.com/forum/#!forum/docker-user)
* Docker on IRC: irc.freenode.net and channel #docker
* [Docker on Twitter](https://twitter.com/docker)
* Get [Docker help](https://stackoverflow.com/search?q=docker) on
  StackOverflow
* [Docker.com](https://www.docker.com/)
