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

<li> Create a Swarm Management Node VM instance.  Associate it with its security group and key pair. Optionally install Docker on the Swarm Management Node to allow you to use the docker cli to send docker commands from the Swarm Management Node to the Docker Nodes for debug purpose.

<li> Create a security group for the Docker nodes.  It containers rules for allowing public access to the SSH port, Ping, and docker auto assigned ports.  The docker auto assigned ports are those ports that docker automatically assigns to containers as there external ports when they are not specifically designated in the docker command.  It also containers a rule for exclusive access to the Docker port from the Swarm Management Node.  Use the Swam Management Node’s public IP.  

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
Install Docker on all the Docker Node instances (http://docs.docker.com/engine/installation/).  
Enable swap cgroup memory limit following those steps from docker  documentation https://docs.docker.com/engine/installation/ubuntulinux/

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
    <b>>apt-get install nfs-kernel-server</b>
     Create a directory that will be used to mount the docker volumes on the docker nodes:
    <b>>sudo mkdir /docker_volumes</b>
     In <b>/etc/hosts/</b> allow access to the docker volume directory to the docker nodes.  For instance:
    <b>/docker_volumes <docker node ip>/24(rw,sync,no_subtree_check,no_root_squash)</b>.
     Start the nfs server:
    <b>>sudo service nfs-kernel-server restart</b>
<li> On the docker nodes install the NFS client and configure it to use the nfs mount point:
    <b>>sudo apt-get install common-nfs</b>
     Mount the docker volumes directory to be backed by nfs:
    <b>>sudo mount -t nfs -o proto=tcp,port=2049 <nfs server ip>:/docker_volumes /var/lib/docker/volumes</b>
     

<li> Start the docker engines daemon to listen on the port that was specified when you created the Docker Nodes security group above.  Optionally, you can allow it to also listen on a linux file socket to simplify debug.  For instance:

    <b>>sudo docker daemon -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 –icc=false </b>
     
     Another alternative is update <b>/etc/default/docker</b> with 
    <b>DOCKER_OPTS="-H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 --icc=false"</b>.  
     And then start docker as a service:
    <b>>sudo service docker restart</b>

<li> Configuring Swarm:
   Environment Variables:
   <ul>
      <li> <b>SWARM_MULTI_TENANT</b>: if set to false vanila swarm is run without multi-tenancy or name scoping.
      <li> <b>SWARM_AUTH_BACKEND:</b> if set to Keystone then Keystone is used to authenticate and authorization docker requests based on the Authorization Token and Tenant ID in their request header. 
      <li> <b>SWARM_MEMBERS_TENANT_ID</b>: contains the tenant id whose members are eligible to use the service. If not set then any valid token tenant id may use the service. SWARM_MEMBERS_TENANT_ID is only valid when SWARM_AUTH_BACKEND is set to Keystone.
      <li> <b>SWARM_ADMIN_TENANT_ID</b>: contains the id of the tenant that may run docker commands as admin. 
      <li> <b>SWARM_FLAVORS_ENFORCED</b>: Flavors enforcement only occurs on docker create.  Docker users can only specify resource combinations that appear in one the predefined flavors. Note there is little flexibility the specified resources must exactly match one of the predefined flavors. Also the set of favors are for all tenants.  By default flavors enforcement are disabled.  However, the swarm administrator can enable it by setting SWARM_FLAVORS_ENFORCED  to true. 
      <li> <b>SWARM_FLAVORS_FILE</b>: Flavors are defined by the swarm administrator in a json file which containers a map of the resource combinations that are enforced.  This is an example:
<sample>
      {   "default":
         {    "Memory": 0,    
              "MemoryReservation": 0,    
              "MemorySwap":         0,    
              "KernelMemory":       0,    
              "CpuShares":          0,    
              "CpuPeriod":          0,    
              "CpuQuota":          0,    
              "BlkioWeight":       0,    
              "OomKillDisable":    false,    
              "MemorySwappiness":  -1,    
              "Privileged":        false,    
              "ReadonlyRootfs":    false   
           },   
         "small":
         {    "Memory": 0,    
              "MemoryReservation": 0,    
              "MemorySwap":         0,    
              "KernelMemory":       0,    
              "CpuShares":          1,    
              "CpuPeriod":          1,    
              "CpuQuota":          0,    
              "BlkioWeight":       0,    
              "OomKillDisable":    false,    
              "MemorySwappiness":  -1,    
              "Privileged":        false,    
              "ReadonlyRootfs":     false   
          } 
     }
</sample>
       By default the json file resides in ./flavors.json, but the administrator can use  SWARM_FLAVORS_FILE to specify another path. The properties file is only read once at initialization.  If the administrator wants to change the flavors it is required to restart swarm.
       
      <li> <b>SWARM_CONFIG</b>: The	Swarm configuration file, authHookConf.json, is a json file that contains the Keystone URL and tenant memory quota limits.
      <code>
      {
         "TenancyLabel":"com.ibm.tenant.0",
         "KeystoneUrl":"http://cloud.lab.fi-ware.org:4730/v2.0/",
         "KeyStoneXAuthToken":"ADMIN",
         "AuthTokenHeader":"X-Auth-Token",
         "quotas":{
                "Memory": 128
          }
      }
      </code>
      If you want to use a different settings make changes to KeystoneURL:<your keystone URL> and/or quota attributes and restart swarm. By default authHookConf.json resides in the same directory from which the swarm binary is started.
Best practice is to set SWARM_CONFIG environment variable that will point to the configuration file. For instance:
     <b>>export SWARM_CONFIG=~/work/src/github.com/docker/swarm/authHookConf.json</b>

    </ul>

<li> Start Multi-Tenant Swarm Manager daemon (without TLS) on the Swarm Management Node.  The Multi-Tenant Swarm docker image resides in the FIWARE Docker Hub repository at [fiware/swarm_multi_tenant](https://hub.docker.com/r/fiware/swarm_multi_tenant/) 
If token discovery is to be used then add the discovery flag, otherwise use the file flag to point to a file with a list of all the Docker Node public ips and docker ports.  For instance:

    ><b>docker run -t -p 2376:2375 -v /tmp/cluster.ipstmp/cluster.ips -e SWARM_AUTH_BACKEND=Keystone -t fiware/swarm_multi_tenant:v0 --debug manage  file:///tmp/cluster.ips</b>    

<li> Test the cluster’s remote connectivity by pinging and sshing to all the instances (including the Swarm Management Node). 

<li> Test whether the Multi-Tenant Swarm Cluster works as expected by using docker commands on your local docker client.  The docker –H flag specifies the Swarm Manager Node and swarm port.  The docker –config specifies the directory where a config.json file is prepared with a valid token and a valid tenantid.  For instance:

    >docker –H tcp://<Swam Manager Node IP>:2376  --config $HOME/dir docker command

See the [FIWARE Docker Container Service Users Guide](https://github.com/fiware-docker/docker-container-service/blob/master/docs/userguide/user-guide.md) for more details on how to use the  service.

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
