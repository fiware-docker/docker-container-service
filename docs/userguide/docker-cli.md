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

Two users are used in the examples, docker-user-1 and docker-user-2.  They are associated with two different tenants as showned by their respective config.json files.
```
$ cat ~/docker-user-1/config.json 
{
	"HttpHeaders": {
            "X-Auth-TenantId": "6885fcb1997446d0b1b2b01e96fc5224",
            "X-Auth-Token": "c4c2452f56994ef8acb8fdf2c2349dcf"
	}
}
$ cat ~/docker-user-2/config.json 
{
	"HttpHeaders": {
            "X-Auth-TenantId": "3850dbe4aeb24e0aa4a13f294fa1b6c1",
            "X-Auth-Token": "6ebe2d0927a44eeb87e951229a765c13"
	}
}

The examples show the two users leveraging docker to manage their docker resources on the FDCS cluster, while FDCS supports multi-tenant isolation and multi-tenant name scoping so that the two users do not interfer with each other.
```
###User defined networks
FDCS supports user defined networks and their management.  Currently it only supports bridge type networks.  In the future it will support overlay networks. Containers on the same network can securely communicate with each other, but those on different networks are isolated from each.  Each network has its own DNS which allows communication between containers on the same network using container names.

####User defined network - ping
In this example we demonstrate FDCS support of user defined bridge networks. It shows  containers on the same network can ping each other using their container name, but those on different networks are isolated from each. It also demonstrates how different tenants are isolated from each other but still can use the same names for their network and containers.  Further we show host port mapping allowing public access to containers residing on user defined network.


docker-user-1 creates and inspects a bridge network called isolated_nw.
```
$ docker --config ~/docker-user-1 network create --driver bridge isolated_nw 
b7b259090b4dd7a7f01426c28ebef51ab07ff87c71433dae4d65b74881e38bb3
$ docker --config ~/docker-user-1 network inspect isolated_nw 
[
    {
        "Name": "isolated_nw",
        "Id": "b7b259090b4dd7a7f01426c28ebef51ab07ff87c71433dae4d65b74881e38bb3",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1/16"
                }
            ]
        },
        "Containers": {},
        "Options": {}
    }
]
```
docker-user-1 runs two containers, container1 and container2, and attaches them to network isolated_nw.  He then inspects network to see that the containers are indeed attached. 
```
$ docker --config ~/docker-user-1 run --net=isolated_nw -itd -p 80 --name=container1 busybox sh -c "httpd;sh"
44bc42c6d2114852617deaf2fa6f1e37717d173b5a6f683b5b256cac0c75b126
$ docker --config ~/docker-user-1 run --net=isolated_nw -itd -p 80 --name=container2 busybox sh -c "httpd;sh"
f7939fb2a7812b56c533733995893b45c9caa794a8d0a90ce045f8baa0faf013
$ docker --config ~/docker-user-1 network inspect isolated_nw 
[
    {
        "Name": "isolated_nw",
        "Id": "b7b259090b4dd7a7f01426c28ebef51ab07ff87c71433dae4d65b74881e38bb3",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1/16"
                }
            ]
        },
        "Containers": {
            "44bc42c6d2114852617deaf2fa6f1e37717d173b5a6f683b5b256cac0c75b126": {
                "Name": "container1",
                "EndpointID": "6b85818b1b2d31686829a688a7ff6c0fbde3177395da49933dcdc8dd39de7bc2",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "f7939fb2a7812b56c533733995893b45c9caa794a8d0a90ce045f8baa0faf013": {
                "Name": "container2",
                "EndpointID": "e1ed906b079283579d80a235506199aeeb07f1b40a2358c97683cead6bc0c628",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]
```
docker-user-1 creates and inspects another bridge network called isolated_nw2.
```
$ docker --config ~/docker-user-1 network create --driver bridge isolated_nw2
ba3df08fd1be678f352d632883bb653ef5459ba72f236ad27e8a80db1a6dfa78
$ docker --config ~/docker-user-1 network inspect isolated_nw2
[
    {
        "Name": "isolated_nw2",
        "Id": "ba3df08fd1be678f352d632883bb653ef5459ba72f236ad27e8a80db1a6dfa78",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1/16"
                }
            ]
        },
        "Containers": {},
        "Options": {}
    }
]
```
docker-user-1 creates another container, container3, and attaches it to network isolated_nw2.
```
$ docker --config ~/docker-user-1 run --net=isolated_nw2 -itd -p 80 --name=container3 busybox sh -c "httpd;sh"
aa11f9313398a1536f2d34b061210653fc84f26c75264a240cbc7962afd4a558
$ docker --config ~/docker-user-1 network inspect isolated_nw2
[
    {
        "Name": "isolated_nw2",
        "Id": "ba3df08fd1be678f352d632883bb653ef5459ba72f236ad27e8a80db1a6dfa78",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1/16"
                }
            ]
        },
        "Containers": {
            "aa11f9313398a1536f2d34b061210653fc84f26c75264a240cbc7962afd4a558": {
                "Name": "container3",
                "EndpointID": "8b67dffffaf0e82f3fb24e59f5c303dc5cffc08e4f4519fcfcb499ab5fc94165",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]

```
docker-user-1 shows its running containers, container1, container2, and container3. Notice that all the container have port 80 mapped to different host urls.
```

$ docker --config ~/docker-user-1  ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                        NAMES
aa11f9313398        busybox             "sh -c httpd;sh"    2 minutes ago       Up 2 minutes        9.148.24.230:32781->80/tcp   rcc-hrl-kvg-588/container3
f7939fb2a781        busybox             "sh -c httpd;sh"    16 minutes ago      Up 16 minutes       9.148.24.231:32780->80/tcp   rcc-hrl-kvg-589/container2
44bc42c6d211        busybox             "sh -c httpd;sh"    16 minutes ago      Up 16 minutes       9.148.24.231:32779->80/tcp   rcc-hrl-kvg-589/container1
```
docker-user-1 demonstates that containers on the same network can communicate with each other over their shared private network, but containers on different networks are isolated from each other.  Notice that container1 can ping container2 since they both reside on the isolated_nw, while container1 can not ping container3 since it does not reside on isolated_nw. We also see that the container name may be used as its DNS name.
```
$ docker --config ~/docker-user-1  exec container1 ping -W 1 -c 3 container2
PING container2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.124 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.039 ms
64 bytes from 172.19.0.3: seq=2 ttl=64 time=0.107 ms

--- container2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.039/0.090/0.124 ms

$ docker --config ~/docker-user-1  exec container1 ping -W 1 -c 3 container3
PING container3 (9.148.41.7): 56 data bytes

--- container3 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss

```
We now demonstate that FDCS supports network tenant isolation and name scoping by introducing a network that is created by another tenant.

docker-user-2 creates a network assigning it the same name already used by docker-user-1, i.e. isolated_nw.  However, FDCS ensures that the two networks are isolated from each other.
```

$ docker --config ~/docker-user-2 network create --driver bridge isolated_nw
57b32fb6742c48f628d66d60acc38dd110c68f6c2a82643d52061c3a2b926364
$ docker --config ~/docker-user-2 network inspect isolated_nw
[
    {
        "Name": "isolated_nw",
        "Id": "57b32fb6742c48f628d66d60acc38dd110c68f6c2a82643d52061c3a2b926364",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1/16"
                }
            ]
        },
        "Containers": {},
        "Options": {}
    }
]
```
docker-user-2 creates and runs two containers assigning the same names used by docker-user-1, i.e. container1 and container3, and attaches it to its isolated_nw.
```
$ docker --config ~/docker-user-2 run --net=isolated_nw -itd -p 80 --name=container1 busybox sh -c "httpd;sh"
70c3164b24de2d412e7f5a4911e43f23550133e81b1470799fc34f59ad014b01
$ docker --config ~/docker-user-2 run --net=isolated_nw -itd -p 80 --name=container3 busybox sh -c "httpd;sh"
385ff253ee27993ed38f2d9f2aceb21f3d01e5c03f96cb3b09a131981b78713d
```
docker-user-2 and docker-user-1 inspect their private isolated_nw networks. docker-user-2's inspect shows container1 and container3 attached to its isolated_nw network. docker-user-1's inspect shows container1 and container2 attached to its isolated_nw network. 
```
$ docker --config ~/docker-user-2 network inspect isolated_nw
[
    {
        "Name": "isolated_nw",
        "Id": "57b32fb6742c48f628d66d60acc38dd110c68f6c2a82643d52061c3a2b926364",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1/16"
                }
            ]
        },
        "Containers": {
            "385ff253ee27993ed38f2d9f2aceb21f3d01e5c03f96cb3b09a131981b78713d": {
                "Name": "container3",
                "EndpointID": "e57d9bec27ab269a339bb430babd4b8391f34d228bce636c7ec76920fbac46ca",
                "MacAddress": "02:42:ac:14:00:03",
                "IPv4Address": "172.20.0.3/16",
                "IPv6Address": ""
            },
            "70c3164b24de2d412e7f5a4911e43f23550133e81b1470799fc34f59ad014b01": {
                "Name": "container1",
                "EndpointID": "f3bf41c1b0376dafaa97babffa875bc4621bd5ea95bb0115ae54e4919c371904",
                "MacAddress": "02:42:ac:14:00:02",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]
$ docker --config ~/docker-user-1 network inspect isolated_nw
[
    {
        "Name": "isolated_nw",
        "Id": "b7b259090b4dd7a7f01426c28ebef51ab07ff87c71433dae4d65b74881e38bb3",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1/16"
                }
            ]
        },
        "Containers": {
            "44bc42c6d2114852617deaf2fa6f1e37717d173b5a6f683b5b256cac0c75b126": {
                "Name": "container1",
                "EndpointID": "6b85818b1b2d31686829a688a7ff6c0fbde3177395da49933dcdc8dd39de7bc2",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "f7939fb2a7812b56c533733995893b45c9caa794a8d0a90ce045f8baa0faf013": {
                "Name": "container2",
                "EndpointID": "e1ed906b079283579d80a235506199aeeb07f1b40a2358c97683cead6bc0c628",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]

```
docker-user-2 and docker-user-1 list their networks. docker-user-2 has one network, isolated_nw. docker-user-1 has two networks, isolated_nw and isolated_nw2.  The two isolated_nw networks are different networks.
```

$ docker --config ~/docker-user-2 network ls
NETWORK ID          NAME                DRIVER
57b32fb6742c        isolated_nw         bridge              
$ docker --config ~/docker-user-1 network ls
NETWORK ID          NAME                DRIVER
b7b259090b4d        isolated_nw         bridge              
ba3df08fd1be        isolated_nw2        bridge

```
We now demonstrate that networks belonging to different networks are isolated from each other. Containers on docker-user-2's isolated_nw can ping each other and docker-user-1's isolated_nw can ping each other. However, containers on docker-user-2's isolated_nw and docker-user-1's isolated_nw can not ping each other.  
```
$ docker --config ~/docker-user-2  exec container1 ping -W 1 -c 3 container3
PING container3 (172.20.0.3): 56 data bytes
64 bytes from 172.20.0.3: seq=0 ttl=64 time=0.128 ms
64 bytes from 172.20.0.3: seq=1 ttl=64 time=0.156 ms
64 bytes from 172.20.0.3: seq=2 ttl=64 time=0.114 ms

--- container3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.114/0.132/0.156 ms

$ docker --config ~/docker-user-1  exec container1 ping -W 1 -c 3 container2
PING container2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.124 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.039 ms
64 bytes from 172.19.0.3: seq=2 ttl=64 time=0.107 ms

--- container2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.039/0.090/0.124 ms

$ docker --config ~/docker-user-1  exec container1 ping -W 1 -c 3 container3
PING container3 (9.148.41.7): 56 data bytes

--- container3 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss

```
We now demonstrate host port mapping to containers on user defined mapping. We show that it allows public access to these containers.

docker-user-2 and docker-user-1 list their containers. docker-user-2 has two containers, container1 and container3. docker-user-1 has three containers, container1, container2, and container3.  Notice that their external URL mapped to port 80 are all different. 
```
$ docker --config ~/docker-user-2 ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                        NAMES
385ff253ee27        busybox             "sh -c httpd;sh"    4 minutes ago       Up 4 minutes        9.148.24.230:32785->80/tcp   rcc-hrl-kvg-588/container3
70c3164b24de        busybox             "sh -c httpd;sh"    4 minutes ago       Up 4 minutes        9.148.24.230:32784->80/tcp   rcc-hrl-kvg-588/container1
$ docker --config ~/docker-user-1 ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                        NAMES
aa11f9313398        busybox             "sh -c httpd;sh"    37 minutes ago      Up 37 minutes       9.148.24.230:32781->80/tcp   rcc-hrl-kvg-588/container3
f7939fb2a781        busybox             "sh -c httpd;sh"    52 minutes ago      Up 52 minutes       9.148.24.231:32780->80/tcp   rcc-hrl-kvg-589/container2
44bc42c6d211        busybox             "sh -c httpd;sh"    52 minutes ago      Up 52 minutes       9.148.24.231:32779->80/tcp   rcc-hrl-kvg-589/container1
```
We now show that it is possible to connect to containers with curl used the host port mappings. 
```
$ user1_container1_URL=$(docker --config ~/docker-user-1 port container1 80)
$ user1_container2_URL=$(docker --config ~/docker-user-1 port container2 80)
$ user1_container3_URL=$(docker --config ~/docker-user-1 port container3 80)
$ user2_container1_URL=$(docker --config ~/docker-user-2 port container1 80)
$ user2_container3_URL=$(docker --config ~/docker-user-2 port container3 80)

$ echo $user1_container1_URL
9.148.24.231:32779
$ echo $user1_container2_URL
9.148.24.231:32780
$ echo $user1_container3_URL
9.148.24.230:32781
$ echo $user2_container1_URL
9.148.24.230:32784
$ echo $user2_container3_URL
9.148.24.230:32785


$ curl -s $user1_container1_URL > /dev/null && echo connected || echo Fail
connected
$ curl -s $user1_container2_URL > /dev/null && echo connected || echo Fail
connected
$ curl -s $user1_container3_URL > /dev/null && echo connected || echo Fail
connected

$ curl -s $user2_container1_URL > /dev/null && echo connected || echo Fail
connected
$ curl -s $user2_container3_URL > /dev/null && echo connected || echo Fail
connected



```



####User defined network - orion and mongodb
In this example we demonstrate user defined networks with FIWARE's orion and mongodb.

User creates a use defined bridge network and then run orion and mongodb containers to communicate over the network:
```
$ network create --driver bridge front
606734595cce235bd70122db0d22dc02068d9c295b7096b7db60bda1074e0504
$ docker network inspect front
[
    {
        "Name": "front",
        "Id": "606734595cce235bd70122db0d22dc02068d9c295b7096b7db60bda1074e0504",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1/16"
                }
            ]
        },
        "Containers": {},
        "Options": {}
    }
]

$ docker run -d --net front --name mongo mongo:3.2 --nojournal
942aaa117c470551afd8df17872b645ef18ca988c91cbd79e1818ccbb3c5bd7e
$ docker run -d -p 1026 --net front --name orion fiware/orion -dbhost mongo
cd1b3bda58237364d7ba48d860392801a2c51cc98eceb3189322a0b465202e21
$ docker network inspect front
[
    {
        "Name": "front",
        "Id": "606734595cce235bd70122db0d22dc02068d9c295b7096b7db60bda1074e0504",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1/16"
                }
            ]
        },
        "Containers": {
            "942aaa117c470551afd8df17872b645ef18ca988c91cbd79e1818ccbb3c5bd7e": {
                "Name": "mongo",
                "EndpointID": "8ed8e6fc6bd8dc31d7194d508b074719d6b5578902d423e613a4ac87b7212511",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "cd1b3bda58237364d7ba48d860392801a2c51cc98eceb3189322a0b465202e21": {
                "Name": "orion",
                "EndpointID": "afff0c6c80c1cc225c299b6456bc149bc238b25477c92700ef13063dab258b79",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                          NAMES
cd1b3bda5823        fiware/orion        "/usr/bin/contextBrok"   9 seconds ago       Up 8 seconds        9.148.24.231:32778->1026/tcp   rcc-hrl-kvg-589/orion
942aaa117c47        mongo:3.2           "/entrypoint.sh --noj"   51 seconds ago      Up 50 seconds       27017/tcp                      rcc-hrl-kvg-589/mongo
```
User verifies that the network is working as expected:
```
$ port=$(docker port orion 1026)
$ curl ${port}/version
{
  "orion" : {
  "version" : "0.28.0-next",
  "uptime" : "0 d, 0 h, 1 m, 56 s",
  "git_hash" : "8c2bfb296628f106bebe5988a48d12efda64dede",
  "compile_time" : "Sat Mar 19 23:24:17 UTC 2016",
  "compiled_by" : "root",
  "compiled_in" : "838a42ae8431"
}
}
$ curl ${port}/v2/entities
[]
$curl ${port}/v2/entities -s -S --header 'Content-Type: application/json' -X POST -d @- <<EOF
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
$ curl ${port}/v2/entities
[{"id":"Room2","type":"Room","pressure":{"type":"Number","value":720,"metadata":{}},"temperature":{"type":"Number","value":23,"metadata":{}}}]

```
### User Defined Volumes
FDCS supports user defined volumes and their management.  Multiple containers can use the same volume in the same time period. This is useful if two containers need access to shared data. For example, if one container writes and the other reads the data. User defined volume data persist even when the containers that reference the volume are removed. Further since FDCS mounts user defined volumes on a NFS mount point the volumes are shared by all the host in the FDCS cluster. Thus, containers on different hosts can easily share data.

In this example we show how two users, docker_user_1 and docker_user_2, leverage user defined volumes to share and persist there data. docker_user_1's volumeA is only accessable to its containers.  docker_user_2's volumeA is only accessable to its containers. 

docker_user-1 creates volumeA. List volume shows the same volume on all hosts in the FDCS cluster. This means no matter  
```
$ docker --config ~/docker-user-1 volume create --name volumeA
volumeA 
$ docker --config ~/docker-user-1 volume ls
DRIVER              VOLUME NAME
local               docker-host-1/volumeA
local               docker-host-2/volumeA
$ docker --config ~/docker-user-1 volume inspect docker-host-1/volumeA
[
    {
        "Name": "volumeA",
        "Driver": "local",
        "Mountpoint": "/var/lib/docker/volumes/volumeA/_data"
    }
]
$ docker --config ~/docker-user-1 run -v volumeA:/data --name con1  busybox sh -c "echo docker_user_1 hello  > /data/file.txt"
$ docker --config ~/docker-user-1 run -v volumeA:/data --name con2  busybox sh -c "cat /data/file.txt"
docker_user_1 hello

$ docker --config ~/docker-user-1 rm con1  
con1
$ docker --config ~/docker-user-1 rm con2
con2
$ docker --config ~/docker-user-1 run -v volumeA:/data --name con2  busybox sh -c "cat /data/file.txt"
docker_user_1 hello

$ docker --config ~/docker-user-2 volume ls
DRIVER              VOLUME NAME
$ docker --config ~/docker-user-2 run -v volumeA:/data  busybox sh -c "cat /data/file.txt"
cat: can't open '/data/file.txt': No such file or directory
$ docker --config ~/docker-user-2 volume create --name volumeA
volumeA
$ docker --config ~/docker-user-2 volume ls
DRIVER              VOLUME NAME
local               docker-host-1/volumeA
local               docker-host-2/volumeA
$ docker --config ~/docker-user-2 run -v volumeA:/data --name con1  busybox sh -c "echo docker_user_2 hello  > /data/file.txt"
$ docker --config ~/docker-user-2 run -v volumeA:/data --name con2  busybox sh -c "cat /data/file.txt"
docker_user_2 hello
$ docker --config ~/docker-user-1 run -v volumeA:/data --name con3 busybox sh -c "cat /data/file.txt"
docker_user_1 hello

$ docker --config ~/docker-user-1  rm -v $(docker --config ~/docker-user-1  ps -a -q)
b5d565bb55bb
c22cd72c4dad
3ff0b33b9476
0b6a4edf6fac
$ docker --config ~/docker-user-2  rm -v $(docker --config ~/docker-user-2  ps -a -q)
768882a1f8ca
7b8f8a0f589a
4434fbd79834
$ docker --config ~/docker-user-1 volume rm volumeA
volumeA
$ docker --config ~/docker-user-2 volume rm volumeA
volumeA
$ docker --config ~/docker-user-1 volume ls
DRIVER              VOLUME NAME
$ docker --config ~/docker-user-2 volume ls
DRIVER              VOLUME NAME
$ 


```

###Container referencing with link and volume-from. 
In this example we use fiware's orion and mongodb to demonstrate container referencing with flags --link and --volume-from. 

User creates a mongodb data volume.
```
docker --config ~/docker-user-1 create -v /data/db --name mongodbdata mongo:3.2 /bin/echo "Data-only container for mongodb."
b9aa0d6cb3d0082f106248cc5afa7978b6ff7921b8a12a402bcf748e1b1ac503

```
User runs mongodb and references the mongodbdata data volume with –volumes-from.
```
docker --config ~/docker-user-1 run -d --volumes-from mongodbdata --name mongodb mongo:3.2 --nojournal
d6ab99615391032e7ed6d9f1b834686d60caee32a933796d6253a9efb201ec15

```
User references the mongodb container with --link.
```
docker --config ~/docker-user-1 run -d --link mongodb  -p :1026 --name orion fiware/orion:latest -dbhost mongodb
52cab61c971806114f68019344734015c1ca7b7e072e1c1a6c51df037c95461d
```
We now access Orion using the port host port assigned to it.
```
port=$(docker --config ~/docker-user-1 port orion 1026)
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
We show that the data base is empty and contains no entities.
```
curl ${port}/v2/entities
[]
```
We create its first entity called Room2 with two attributes temperature and pressure.
```
curl ${port}/v2/entities -s -S --header 'Content-Type: application/json' -X POST -d @- <<EOF
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
We now demonstate that data persists in mongodbdata by removing all containers  accept mongodbdata  and then run again.

```
docker --config ~/docker-user-1 rm -fv orion
docker --config ~/docker-user-1 rm -fv mongodb
docker --config ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
1fdec401e212        mongo:3.2           "/entrypoint.sh /bin/"   41 minutes ago      Created                                 rcc-hrl-kvg-588/mongodbdata

```
User brings orion and mongodb up again and shows that the data previously injected into the data base is still there.
````
$ docker run -d --volumes-from mongodbdata --name mongodb mongo:3.2 --nojournal
ebc4c11c273a010437cf6d5a02442c327e0e655658d2132b0cebf212750bae11
$ docker run -d --link mongodb  -p :1026 --name orion fiware/orion:latest -dbhost mongodb
70ee33e22b44b948e4bfbc7ebd7a7d29abafd0f68a56376c91ca4161b98b435b
$ port=$(docker port orion 1026)
$ curl ${port}/v2/entities/Room2 | python -mjson.tool
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
