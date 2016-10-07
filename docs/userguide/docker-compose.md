#Docker Compose

Once you prepare your docker client as described in the user guide's [Quick Start](https://github.com/fiware-docker/docker-container-service/blob/master/docs/userguide/user-guide.md/user-guide.md##Quick Start) you can use [Docker Compose](https://docs.docker.com/compose/). We support Docker-Compose 1.6.2 and above.

Note: docker-compose does not support the docker cli --config flag, so the ~/.docker/config.json must contain headers X-Auth-Token and X-Auth-TenantId. Likewise, docker-compose does not support the docker cli -H flag so the environment variable must be set to tcp://docker.lab.fiware.org:2376.

The [docker compose command-line](https://docs.docker.com/compose/reference/) is supported.

These docker compose commands are supported:
- build: NA
- config: NA
- create: supported
- down: supported
- events: not supported
- kill: supported
- logs: supported
- pause: supported
- port: supported
- ps: supported
- pull: supported
- restart: supported
- rm: supported
- run: supported
- scale: supported
- start: supported
- stop: supported
- unpause: supported
- up: supported

There are two versions of the [docker-compose.yml file](https://docs.docker.com/compose/compose-file/) format. Version 1 is the legacy format, which does not support the volume_driver or networks tag.  Most of the version 1 and 2 docker-compose tags are supported, but they may be restricted in a similar manner to the restriction applied to the docker CLI. For instance, the volumes tag is supported, but FDCS does not permit reference to the host filesystem. User defined volumes can be referenced in the in the volumes tag. 

## Examples
### Orion and Mongodb
This docker-compose example demonstrates a docker service assembled from docker images **FIWARE/orion** and **mongo:3.2**.  The DB is made persistent on a user define NFS volume called **mongodata**. The orion container communicates with the mongo container over a user defined overlay network called **front**.

This the docker-compose.yml:
```
/orion$ cat docker-compose.yml 
version: '2'
networks:
  front:
     driver: "overlay"
volumes:
  mongodata:
#     external: true
     driver: "nfs"
services: 
   mongo:
     image: mongo:3.2
     command: --nojournal
     networks:
      - front
     volumes:
      - mongodata:/data/db

   orion:
     image: fiware/orion
     ports:
       - "1026"
     networks:     
      - front

     command: -dbhost mongo
```
Notice that it creates a user defined overlay network called **front** and a user defined NFS volume called **mongodata**. Both containers, **orion** and **mongo**, reference the *front* network. The mongo container references the *mongodata* volume and mounts its /data/db on the volume. The *mongodata* volume could have been created outside of the docker-compose file using docker-cli; in that case the *external* tag would have been used. 

Initially, there are no running containers, no user defined volumes  and no user defined networks.
```
/orion$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
docker-user-1@rcc-hrl-kvg-558:~/orion$ docker volume ls
DRIVER              VOLUME NAME
docker-user-1@rcc-hrl-kvg-558:~/orion$ docker network ls
NETWORK ID          NAME                DRIVER
```

We launch the service with docker-compose up -d

```
/orion$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
docker-user-1@rcc-hrl-kvg-558:~/orion$ docker volume ls
DRIVER              VOLUME NAME
docker-user-1@rcc-hrl-kvg-558:~/orion$ docker network ls
NETWORK ID          NAME                DRIVER
docker-user-1@rcc-hrl-kvg-558:~/orion$ docker-compose up -d
Creating network "orion_front" with driver "overlay"
Creating volume "orion_mongodata" with nfs driver
Creating orion_orion_1
Creating orion_mongo_1

```
It creates an overlay network (orion_front) and the user defined nfs volume (orion_mongodata), and two contains (orion_orion_1 and orion_mongo_1).
docker_compose prefixes the containers with the directory name from which it launches the docker-compose file.  It also appends a number to the container names that is used when it scales out a service.
```
~/orion$ docker-compose ps
    Name                   Command               State               Ports              
---------------------------------------------------------------------------------------
orion_mongo_1   /entrypoint.sh --nojournal       Up      27017/tcp                      
orion_orion_1   /usr/bin/contextBroker -fg ...   Up      130.206.119.32:32768->1026/tcp 

docker-user-1@rcc-hrl-kvg-558:~/orion$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
40e14092eddf        fiware/orion        "/usr/bin/contextBrok"   8 minutes ago       Up 8 minutes        130.206.119.32:32769->1026/tcp   docker-host-3/orion_orion_1
59a647da904e        mongo:3.2           "/entrypoint.sh --noj"   8 minutes ago       Up 8 minutes        27017/tcp                        docker-host-2/orion_mongo_1

```
Notice that containers are running on different docker hosts. The orion container is running on docker-host-3 and the mongo container is running on docker-host-2. They can interact with each other over their own private overlay network. Notice also that the orion container's port 1026 is mapped to the docoker-host-3's port 32768.  Later we will run curl commands to orion using the host port mapping.

We look at the volume that was created.  Notice that docker-compose also prefixes the volume name with directory name from which the docker-compose file was launched.  Two volume entries appear because the same nfs volume is mounted to both hosts in the cluster.
```
/orion$ docker volume ls
DRIVER              VOLUME NAME
nfs                 orion_mongodata
nfs                 orion_mongodata
```
We look at the network that was created.  Notice that docker-compose also prefixes the network name with directory name from which the docker-compose file was launched.  
```
/orion$ docker network ls
NETWORK ID          NAME                DRIVER
1b651ad80f4b        orion_front         overlay 
/orion$ docker network inspect orion_front
[
    {
        "Name": "orion_front",
        "Id": "1b651ad80f4b9567626faba94accdf74aa77217d91258105525d56a2d5907426",
        "Scope": "global",
        "Driver": "overlay",
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1/24"
                }
            ]
        },
        "Containers": {
            "c67fbd499135cf20aa301a5e9c9b8611330a64cf1ac5315ca47fde5b5a4ecf50": {
                "Name": "orion_orion_1",
                "EndpointID": "4d5111cb9303e61bf28f3a5209e788d2e33c9d3f1a5c29ebdfcce8a1503ddc9e",
                "MacAddress": "02:42:0a:00:00:02",
                "IPv4Address": "10.0.0.2/24",
                "IPv6Address": ""
            },
            "cba077da484f4b5f9ff6b47721810e7660dce5979ed84636573626f28d97d533": {
                "Name": "orion_mongo_1",
                "EndpointID": "fde05d125ba6971218eff9b5092877f7c6e47b8baaa67d156be83957b43baeab",
                "MacAddress": "02:42:0a:00:00:03",
                "IPv4Address": "10.0.0.3/24",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]

```

We fetch orion's host port mapped URL with the docker-compose port command so that we can use curl to get and post entities from/to orion and the mongo DB.
```
/orion$ port=$(docker-compose port orion 1026)
```
We show that DB is initially empty. 
```
/orion$ curl $port/v2/entities
[]
```
We add to the DB entity Room2 with attributes temperature and pressure and validates that the entity is indeed added to the DB. 
```
/orion$ curl ${port}/v2/entities -s -S --header 'Content-Type: application/json' -X POST -d @- <<EOF
{
  "id": "Room2",
  "type": "Room",
  "temperature": {
    "value": 23,
    "type": "Number"
  },
  "pressure": {
    "value": 720,
    "type": "Number"
  }
}
EOF
/orion$ curl $port/v2/entities
[{"id":"Room2","type":"Room","pressure":{"type":"Number","value":720,"metadata":{}},"temperature":{"type":"Number","value":23,"metadata":{}}}]
```
We now show that the data is persistent by removing the containers with the docker-compose down command.  The containers and network are removed but the volume, mongodata, remains untouched.
```
docker-user-1@rcc-hrl-kvg-558:~/orion$ docker-compose down
Stopping orion_mongo_1 ... done
Stopping orion_orion_1 ... done
Removing orion_mongo_1 ... done
Removing orion_orion_1 ... done
Removing network orion_front
docker-user-1@rcc-hrl-kvg-558:~/orion$ docker network ls
NETWORK ID          NAME                DRIVER
docker-user-1@rcc-hrl-kvg-558:~/orion$ docker volume ls
DRIVER              VOLUME NAME
nfs                 orion_mongodata
nfs                 orion_mongodata

```
We now launch the service again and show that the DB still containers Room2 and  its attributes.

```
docker-user-1@rcc-hrl-kvg-558:~/orion$ docker-compose up -d
Creating network "orion_front" with driver "overlay"
Creating volume "orion_mongodata" with nfs driver
Creating orion_orion_1
Creating orion_mongo_1
docker-user-1@rcc-hrl-kvg-558:~/orion$ docker network ls
NETWORK ID          NAME                DRIVER
d92818b2e5c0        orion_front         overlay             
docker-user-1@rcc-hrl-kvg-558:~/orion$ port=$(docker-compose port orion 1026)
docker-user-1@rcc-hrl-kvg-558:~/orion$ curl $port/v2/entities
[{"id":"Room2","type":"Room","pressure":{"type":"Number","value":720.000000},"temperature":{"type":"Number","value":23.000000}}]docker-user-1@rcc-hrl-kvg-558:~/orion$ 

```

