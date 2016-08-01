#Docker CLI
Once you prepare your docker client as described in the user guide's [Quick Start](./user-guide.md##Quick Start) you can use the [Docker CLI](https://docs.docker.com/engine/reference/commandline/cli/). We support docker 1.10.3 and above. 


Most of the commands to manage your containers are supported. But they are limited to containers that belonging to the tenant specified in your config.json file.  For instance, ps will only list containers belonging to the specified tenant.

These docker cli commands are supported:

- attach
- create (see flag restrictions below)
- diff
- cp
- exec
- inspect
- kill
- logs
- network connect (currently not supported but we are working on it)
- network create (currently not supported but we are working on it)
- network disconnect(currently not supported but we are working on it)
- network inspect (currently not supported but we are working on it)
- network ls (currently not supported but we are working on it)
- network rm (currently not supported but we are working on it)
- pause
- port
- ps
- pull 
- rename (currently not supported but we are working on it)
- restart
- rm
- run (see flag restrictions below)
- search
- start
- stats
- stop
- top
- update
- unpause
- version
- volume create  
- volume inspect 
- volume ls 
- volume rm 
- wait (currently not supported but we are working on it)

These docker run flags are supported, but there may be some restrictions:

- a, attach
- d, detach
- e, env  (note affinity and constraints are suppoted) 
- entrypint
- expose
- hostname
- help
- i, interactive
- l, label
- m (must be less than or equal to the default tenant quota)
- memory-reservation (must be less than or equal to the default tenant quota)
- memory-swap (must be less than or equal to the default tenant quota)
- memory-swappiness ?
- name
- net  ?
- oom-kill-disable ?
- P, publish-all
- p, --publish (external host port not allowed)
- restart 
- rm
- stop-signal
- t, tty
- v, volume ((local host directory not allowed)
- volumes-from
- w, workdir

There docker run and create flags are not supported:

- add-hosts
- blkio-weight
- cpu-shares
- cap-add
- cap-drop
- cgroup-parent
- cidfile
- cpu-period
- cpu-quota
- cpuset-cpus
- cpuset-mems
- device
- disable-content-trust
- dns ?
- dns-opt ?
- dns-search ? 
- env-file ?
- group-add
- ipc
- kernel-memory
- label-file
- log-driver
- log-opt
- lxc-conf
- mac-address
- net ?
- oom-kill-disable
- pid
- privileged
- read-only
- restart ?
- security-opt
- sig-proxy
- u, user 
- ulimit
- uts



These docker cli commands are not supported:

- daemon
- buid
- commit
- cp
- events
- export
- history
- images
- import
- info
- load
- login
- logout
- push
- save
- tag

## Examples
###Orion
Create Data Volume
```
docker --config $DOCKER_CONFIG1 create -v /data/db --name mongodbdata mongo:3.2 /bin/echo "Data-only container for mongodb."
b9aa0d6cb3d0082f106248cc5afa7978b6ff7921b8a12a402bcf748e1b1ac503

```
Use –volumes-from to reference data volume
```
docker --config $DOCKER_CONFIG1 run -d --volumes-from mongodbdata --name mongodb mongo:3.2 --nojournal
d6ab99615391032e7ed6d9f1b834686d60caee32a933796d6253a9efb201ec15

```
Reference data base container with --link
```
docker --config $DOCKER_CONFIG1 run -d --link mongodb  -p :1026 --name orion fiware/orion:latest -dbhost mongodb
52cab61c971806114f68019344734015c1ca7b7e072e1c1a6c51df037c95461d
```
Access orion
port=$(docker --config $DOCKER_CONFIG1 port orion 1026)
curl ${port}/version
{
  "orion" : {
  "version" : "0.28.0-next",
  "uptime" : "0 d, 0 h, 4 m, 37 s",
  "git_hash" : "8c2bfb296628f106bebe5988a48d12efda64dede",
  "compile_time" : "Sat Mar 19 23:24:17 UTC 2016",
  "compiled_by" : "root",
  "compiled_in" : "838a42ae8431"
}
}
```
curl ${port}/v2/entities
[]
curl ${port}/entities -s -S --header 'Content-Type: application/json' -X POST -d @- <<EOF
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
curl ${port}/v2/entities/Room2 -s -S --header 'Accept: application/json' | python -mjson.tool
curl ${port}/v2/entities/Room2 | python -mjson.tool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   140  100   140    0     0  10150      0 --:--:-- --:--:-- --:--:-- 10769
{
    "id": "Room2",
    "pressure": {
        "metadata": {},
        "type": "Number",
        "value": 720
    },
    "temperature": {
        "metadata": {},
        "type": "Number",
        "value": 23
    },
    "type": "Room"
}
```
Demonstate that data persists in mongodbdata.
Remove everything accept mongodbdata  and then run again.
```
docker --config $DOCKER_CONFIG1 rm -fv orion
docker --config $DOCKER_CONFIG1 rm -fv mongodb
docker --config ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
1fdec401e212        mongo:3.2           "/entrypoint.sh /bin/"   41 minutes ago      Created                                 rcc-hrl-kvg-588/mongodbdata

```
Bring orion and mongodb up again.
````
nagin@rcc-hrl-kvg-558:~/compose_test/orion$ docker run -d --volumes-from mongodbdata --name mongodb mongo:3.2 --nojournal
ebc4c11c273a010437cf6d5a02442c327e0e655658d2132b0cebf212750bae11
nagin@rcc-hrl-kvg-558:~/compose_test/orion$ docker run -d --link mongodb  -p :1026 --name orion fiware/orion:latest -dbhost mongodb
70ee33e22b44b948e4bfbc7ebd7a7d29abafd0f68a56376c91ca4161b98b435b
nagin@rcc-hrl-kvg-558:~/compose_test/orion$ port=$(docker port orion 1026)
nagin@rcc-hrl-kvg-558:~/compose_test/orion$ curl ${port}/v2/entities/Room2 | python -mjson.tool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   140  100   140    0     0  15140      0 --:--:-- --:--:-- --:--:-- 15555
{
    "id": "Room2",
    "pressure": {
        "metadata": {},
        "type": "Number",
        "value": 720
    },
    "temperature": {
        "metadata": {},
        "type": "Number",
        "value": 23
    },
    "type": "Room"
}
```

