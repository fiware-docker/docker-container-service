#Docker Compose

Once you prepare your docker client as described in the user guide's [Quick Start](./user-guide.md##Quick Start) you can use the [Docker Compose](https://docs.docker.com/compose/).  We support Docker-Compose 1.6.2 and above.

Note: docker-compose does not support the docker cli --config flag, so the ~/.docker/config.json must container headers X-Auth-Token and X-Auth-TenantId. Likewise, docker-compose does not support the docker cli -H flag so the environment variable must be set to tcp://docker.lab.fiware.org:2376.

The [docker compose cli](https://docs.docker.com/compose/reference/) is supported.

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

There are two versions of the [docker-compose.yml file](https://docs.docker.com/compose/compose-file/) format. Version 1 is the legacy format, which does not support the volume_driver or networks tag.  Currently FDCS does not support Version 2. Until version 2 is supported you won't be able declare named networks. Most of the version 2 docker-compose tags are supported, but they may be restricted in a similar manner to the restriction applied to the docker CLI. For instance, the volumes tag is supported, but FDCS does not permit reference to the host filesystem. User defined volumes can be referenced in the in the volumes tag. 

## Examples
### Orion and Mongodb
This docker-compose examples demonstrated a docker service assembled from FIWARE orion and Mongodb.  The DB is made persistent on user define volume, mongodata.

This the docker-compose.yml:
```
/orion$ cat docker-compose.yml
 mongo:
   image: mongo:3.2
   volumes:
      - mongodata:/data/db
   command: --nojournal
 orion:
   image: fiware/orion
   links:
     - mongo
   ports:
     - "1026"
   command: -dbhost mongo
```
We launch the service with docker-compose up -d

```
/orion$ docker volume ls
DRIVER              VOLUME NAME
nagin@rcc-hrl-kvg-558:~/compose_test/orion$ docker-compose up -d
Creating orion_mongo_1
Creating orion_orion_1
```
It creates two contains and the user defined volume mongodata.
```
/orion$ docker-compose ps
    Name                   Command               State              Ports             
-------------------------------------------------------------------------------------
orion_mongo_1   /entrypoint.sh --nojournal       Up      27017/tcp                    
orion_orion_1   /usr/bin/contextBroker -fg ...   Up      9.148.24.230:32796->1026/tcp 

/orion$ docker volume ls
DRIVER              VOLUME NAME
local               rcc-hrl-kvg-588/mongodata

```

We fetch orion's host port mapped URL with the docker-compose port command so that we can use curl to get and post entities from/to orion and the mongo DB.
```
/orion$ port=$(docker-compose port orion 1026)
```
We show that DB is intitially empty. 
```
/orion$ curl $port/v2/entities
[]
```
We add to the DB enitity Room2 with attributes temperature and pressure and validates that the entity is indeed added to the DB. 
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
We now show that the data is persistent by removing the containers with the docker-compose down command.  The containers are removed but mongodata remains untouched.
```
/orion$ docker-compose down
Stopping orion_orion_1 ... done
Stopping orion_mongo_1 ... done
Removing orion_orion_1 ... done
Removing orion_mongo_1 ... done
nagin@rcc-hrl-kvg-558:~/compose_test/orion$ docker-compose ps
Name   Command   State   Ports 
------------------------------
/orion$ docker volume ls
DRIVER              VOLUME NAME
local               rcc-hrl-kvg-588/mongodata
```
We now launch the service again and show that the DB still containers Room2 and  its attributes.

```
nagin@rcc-hrl-kvg-558:~/compose_test/orion$ docker-compose up -d
Creating orion_mongo_1
Creating orion_orion_1
nagin@rcc-hrl-kvg-558:~/compose_test/orion$ port=$(docker-compose port orion 1026)
nagin@rcc-hrl-kvg-558:~/compose_test/orion$ curl $port/v2/entities
[{"id":"Room2","type":"Room","pressure":{"type":"Number","value":720,"metadata":{}},"temperature":{"type":"Number","value":23,"metadata":{}}}]

```

