#Docker CLI
Once you prepare your docker client as described in the user guide's [Quick Start](https://github.com/fiware-docker/docker-container-service/blob/master/docs/userguide/user-guide.md/user-guide.md##Quick Start) you can use the [Docker CLI](https://docs.docker.com/engine/reference/commandline/cli/). We support docker 1.10.3 and above. 


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
- network connect
- network create
- network disconnect
- network inspect
- network ls
- network rm
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
- net  
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
FDCS supports user defined overlay networks and their management. Containers on the same overlay network can securely communicate with each other, but those on different networks are isolated from each.  Each network has its own DNS which allows communication between containers on the same network using container names.  The containers may be running on different docker hosts in the FDCS cluster.

####User defined network - ping
In this example we demonstrate FDCS support of user defined overlay networks. It shows  containers on the same network can ping each other using their container name, but those on different networks are isolated from each. It also demonstrates how different tenants are isolated from each other but still can use the same names for their networks and containers.  Further we show host port mapping allowing public access to containers residing on user defined network.


docker-user-1 creates and inspects a bridge network called isolated_nw.
```
$ docker --config ~/docker-user-1 network ls
NETWORK ID          NAME                DRIVER
$ docker --config ~/docker-user-1 network create --driver overlay isolated_nw
38b3928cea20e80e1309d240c2ff7185fdb4ae39962cb235d642ccaf01d9b420
$ docker --config ~/docker-user-1 network ls
NETWORK ID          NAME                DRIVER
38b3928cea20        isolated_nw         overlay             
$ docker --config ~/docker-user-1 network inspect isolated_nw
[
    {
        "Name": "isolated_nw",
        "Id": "38b3928cea20e80e1309d240c2ff7185fdb4ae39962cb235d642ccaf01d9b420",
        "Scope": "global",
        "Driver": "overlay",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1/24"
                }
            ]
        },
        "Containers": {},
        "Options": {}
    }
]

```
docker-user-1 runs two containers, container1 and container2, and attaches them to network isolated_nw.  He then inspects the network to see that the containers are indeed attached.  
```
$ docker --config ~/docker-user-1 run --net=isolated_nw -itd -p 80 --name=container1 busybox httpd -f -p 80
722ea834daf52c641f1606ad594dce5def56815c421c367fe28d5d3eb33aec2d
$ docker --config ~/docker-user-1 run --net=isolated_nw -itd -p 80 --name=container2 busybox httpd -f -p 80
7b17c249ac4bbc1211c9a048e0775b18cba5435858b3773086972806cf25cc36
$ docker --config ~/docker-user-1 ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                          NAMES
7b17c249ac4b        busybox             "httpd -f -p 80"    14 seconds ago      Up 11 seconds       130.206.119.32:32770->80/tcp   docker-host-3/container2
722ea834daf5        busybox             "httpd -f -p 80"    43 seconds ago      Up 40 seconds       130.206.119.28:32771->80/tcp   docker-host-2/container1

$ docker --config ~/docker-user-1 network inspect isolated_nw
[
    {
        "Name": "isolated_nw",
        "Id": "38b3928cea20e80e1309d240c2ff7185fdb4ae39962cb235d642ccaf01d9b420",
        "Scope": "global",
        "Driver": "overlay",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1/24"
                }
            ]
        },
        "Containers": {
            "722ea834daf52c641f1606ad594dce5def56815c421c367fe28d5d3eb33aec2d": {
                "Name": "container1",
                "EndpointID": "2b98b5212f15c39a370a4895956ca3c7d35e8223f241ce5b151cd04a9f428572",
                "MacAddress": "02:42:0a:00:00:02",
                "IPv4Address": "10.0.0.2/24",
                "IPv6Address": ""
            },
            "7b17c249ac4bbc1211c9a048e0775b18cba5435858b3773086972806cf25cc36": {
                "Name": "container2",
                "EndpointID": "1e9bcc0c6261509a6a644cdc6df0e8b01adafbaac9d338ec6164137a590f8b77",
                "MacAddress": "02:42:0a:00:00:03",
                "IPv4Address": "10.0.0.3/24",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]

```
docker-user-1 shows that two containers can ping each other using their container names or their ip address as shown in the network inspect.   
```

$ docker --config ~/docker-user-1 exec container1 ping -c 2 container2
PING container2 (10.0.0.3): 56 data bytes
64 bytes from 10.0.0.3: seq=0 ttl=64 time=24.344 ms
64 bytes from 10.0.0.3: seq=1 ttl=64 time=24.144 ms

--- container2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 24.144/24.244/24.344 ms
docker-user-1@rcc-hrl-kvg-558:~$ docker exec container2 ping -c 2 container1
PING container1 (10.0.0.2): 56 data bytes
64 bytes from 10.0.0.2: seq=0 ttl=64 time=24.267 ms
64 bytes from 10.0.0.2: seq=1 ttl=64 time=24.412 ms

--- container1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 24.267/24.339/24.412 ms


docker-user-1@rcc-hrl-kvg-558:~$ docker exec container1 ping -c 2 10.0.0.3
PING 10.0.0.3 (10.0.0.3): 56 data bytes
64 bytes from 10.0.0.3: seq=0 ttl=64 time=24.166 ms
64 bytes from 10.0.0.3: seq=1 ttl=64 time=24.328 ms

--- 10.0.0.3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 24.166/24.247/24.328 ms


docker-user-1@rcc-hrl-kvg-558:~$ docker exec container2 ping -c 2 10.0.0.2
PING 10.0.0.2 (10.0.0.2): 56 data bytes
64 bytes from 10.0.0.2: seq=0 ttl=64 time=24.409 ms
64 bytes from 10.0.0.2: seq=1 ttl=64 time=24.280 ms

--- 10.0.0.2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 24.280/24.344/24.409 ms


```
docker-user-1 creates and inspects another bridge network called isolated_nw2.
```
$ docker --config ~/docker-user-1 network create --driver overlay isolated_nw2
ee829ebc5f528b0c4e726f27f6c77f24da165ec764bc5b5d42a1d561b6ca5a03
docker-user-1@rcc-hrl-kvg-558:~$ docker network ls
NETWORK ID          NAME                DRIVER
38b3928cea20        isolated_nw         overlay             
ee829ebc5f52        isolated_nw2        overlay             
docker-user-1@rcc-hrl-kvg-558:~$ docker network inspect isolated_nw2
[
    {
        "Name": "isolated_nw2",
        "Id": "ee829ebc5f528b0c4e726f27f6c77f24da165ec764bc5b5d42a1d561b6ca5a03",
        "Scope": "global",
        "Driver": "overlay",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.1.0/24",
                    "Gateway": "10.0.1.1/24"
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
$ docker --config ~/docker-user-1 run --net=isolated_nw2 -itd -p 80 --name=container3 busybox httpd -f -p 80
3ff7da20a937272129b0a7159bcbf7092536d38866b39f4ea047878bcfba171b
docker-user-1@rcc-hrl-kvg-558:~$ docker network inspect isolated_nw2
[
    {
        "Name": "isolated_nw2",
        "Id": "ee829ebc5f528b0c4e726f27f6c77f24da165ec764bc5b5d42a1d561b6ca5a03",
        "Scope": "global",
        "Driver": "overlay",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.1.0/24",
                    "Gateway": "10.0.1.1/24"
                }
            ]
        },
        "Containers": {
            "3ff7da20a937272129b0a7159bcbf7092536d38866b39f4ea047878bcfba171b": {
                "Name": "container3",
                "EndpointID": "efee5206d8b50ba7421873bea9fabfb470a2a2bf115ba869e6feb5c838349a6a",
                "MacAddress": "02:42:0a:00:01:02",
                "IPv4Address": "10.0.1.2/24",
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
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                          NAMES
3ff7da20a937        busybox             "httpd -f -p 80"    3 minutes ago       Up 3 minutes        130.206.119.28:32772->80/tcp   docker-host-2/container3
7b17c249ac4b        busybox             "httpd -f -p 80"    16 minutes ago      Up 16 minutes       130.206.119.32:32770->80/tcp   docker-host-3/container2
722ea834daf5        busybox             "httpd -f -p 80"    17 minutes ago      Up 16 minutes       130.206.119.28:32771->80/tcp   docker-host-2/container1

```
docker-user-1 demonstates that containers on the same network can communicate with each other over their shared private network, but containers on different networks are isolated from each other.  Notice that container1 can ping container2 since they both reside on the isolated_nw, while container1 can not ping container3 since it does not reside on isolated_nw. 
```
$ docker --config ~/docker-user-1 exec container1 ping -c 2 container2
PING container2 (10.0.0.3): 56 data bytes
64 bytes from 10.0.0.3: seq=0 ttl=64 time=24.685 ms
64 bytes from 10.0.0.3: seq=1 ttl=64 time=24.176 ms

--- container2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 24.176/24.430/24.685 ms

$ docker --config ~/docker-user-1 --config ~/docker-user-1 exec container1 ping -c 2 container3
ping: bad address 'container3'

$ docker exec container1 --config ~/docker-user-1 ping -c 2 10.0.1.2
PING 10.0.1.2 (10.0.1.2): 56 data bytes

--- 10.0.1.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

```
We now demonstate that FDCS supports network tenant isolation and name scoping by introducing a network that is created by another tenant.

docker-user-2 creates a network assigning it the same name already used by docker-user-1, i.e. isolated_nw.  However, FDCS ensures that the two networks are isolated from each other.
```
$ docker --config ~/docker-user-2 network ls
NETWORK ID          NAME                DRIVER
$ docker --config ~/docker-user-2 network create --driver overlay isolated_nw
9e2c8cb747622eb6993b8a2b6aa914074df7c64c7d24cb95681a8a3f2e3925eb
$ docker --config ~/docker-user-2 network inspect isolated_nw
[
    {
        "Name": "isolated_nw",
        "Id": "9e2c8cb747622eb6993b8a2b6aa914074df7c64c7d24cb95681a8a3f2e3925eb",
        "Scope": "global",
        "Driver": "overlay",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.2.0/24",
                    "Gateway": "10.0.2.1/24"
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
$ docker --config ~/docker-user-2 run --net=isolated_nw -itd -p 80 --name=container1 busybox httpd -f -p 80
67a15cd9f94d8b39c9f8c0f255356657443dbed13c9177308bbdeec490fb6805
$ docker --config ~/docker-user-2 run --net=isolated_nw -itd -p 80 --name=container3 busybox httpd -f -p 80
55761092f6f9e93ea42e3fafc481772658d239ad6b23e73379f754d49ddab337

```
docker-user-2 and docker-user-1 inspect their private isolated_nw networks. docker-user-2's inspect shows container1 and container3 attached to its isolated_nw network. docker-user-1's inspect shows container1 and container2 attached to its isolated_nw network. 
```
$ docker --config ~/docker-user-2 network inspect isolated_nw
[
    {
        "Name": "isolated_nw",
        "Id": "9e2c8cb747622eb6993b8a2b6aa914074df7c64c7d24cb95681a8a3f2e3925eb",
        "Scope": "global",
        "Driver": "overlay",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.2.0/24",
                    "Gateway": "10.0.2.1/24"
                }
            ]
        },
        "Containers": {
            "67a15cd9f94d8b39c9f8c0f255356657443dbed13c9177308bbdeec490fb6805": {
                "Name": "container1",
                "EndpointID": "8fd66d66668290f0459a6870b7bdab3ee39b6a3a02da2895c4a30f90416d9a5b",
                "MacAddress": "02:42:0a:00:02:02",
                "IPv4Address": "10.0.2.2/24",
                "IPv6Address": ""
            },
            "ep-b80c1415d279e27d646dce898dfe35733448413a33675b9f1cfcb7b72f493f00": {
                "Name": "container3",
                "EndpointID": "b80c1415d279e27d646dce898dfe35733448413a33675b9f1cfcb7b72f493f00",
                "MacAddress": "02:42:0a:00:02:03",
                "IPv4Address": "10.0.2.3/24",
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
        "Id": "38b3928cea20e80e1309d240c2ff7185fdb4ae39962cb235d642ccaf01d9b420",
        "Scope": "global",
        "Driver": "overlay",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1/24"
                }
            ]
        },
        "Containers": {
            "722ea834daf52c641f1606ad594dce5def56815c421c367fe28d5d3eb33aec2d": {
                "Name": "container1",
                "EndpointID": "2b98b5212f15c39a370a4895956ca3c7d35e8223f241ce5b151cd04a9f428572",
                "MacAddress": "02:42:0a:00:00:02",
                "IPv4Address": "10.0.0.2/24",
                "IPv6Address": ""
            },
            "7b17c249ac4bbc1211c9a048e0775b18cba5435858b3773086972806cf25cc36": {
                "Name": "container2",
                "EndpointID": "1e9bcc0c6261509a6a644cdc6df0e8b01adafbaac9d338ec6164137a590f8b77",
                "MacAddress": "02:42:0a:00:00:03",
                "IPv4Address": "10.0.0.3/24",
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
9e2c8cb74762        isolated_nw         overlay             
$ docker --config ~/docker-user-1 network ls
NETWORK ID          NAME                DRIVER
b7b259090b4d        isolated_nw         overlay              
ba3df08fd1be        isolated_nw2        overlay

```
We now demonstrate that networks belonging to different networks are isolated from each other. Containers on docker-user-2's isolated_nw can ping each other and docker-user-1's isolated_nw can ping each other. However, containers on docker-user-2's isolated_nw and docker-user-1's isolated_nw can not ping each other.  
```
$ docker --config ~/docker-user-2  exec container1 ping -c 2 container3
PING container3 (10.0.2.3): 56 data bytes
64 bytes from 10.0.2.3: seq=0 ttl=64 time=24.824 ms
64 bytes from 10.0.2.3: seq=1 ttl=64 time=24.195 ms

--- container3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 24.195/24.509/24.824 ms

$ docker --config ~/docker-user-2  exec container1 ping -c 2 10.0.2.3

PING 10.0.2.3 (10.0.2.3): 56 data bytes
64 bytes from 10.0.2.3: seq=0 ttl=64 time=24.571 ms
64 bytes from 10.0.2.3: seq=1 ttl=64 time=24.148 ms

--- 10.0.2.3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 24.148/24.359/24.571 ms

$ docker --config ~/docker-user-1 exec container1 ping -c 2 container3
ping: bad address 'container3'

$ docker --config ~/docker-user-1  exec container1 ping -c 2 10.0.2.3
PING 10.0.2.3 (10.0.2.3): 56 data bytes

--- 10.0.2.3 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

$ docker --config ~/docker-user-1 exec container1 ping -c 2 10.0.2.3
PING 10.0.2.3 (10.0.2.3): 56 data bytes

--- 10.0.2.3 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

$ docker --config ~/docker-user-1 exec container1 ping -c 2 container2
PING container2 (10.0.0.3): 56 data bytes
64 bytes from 10.0.0.3: seq=0 ttl=64 time=24.400 ms
64 bytes from 10.0.0.3: seq=1 ttl=64 time=24.219 ms

--- container2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 24.219/24.309/24.400 ms

```
We now demonstrate host port mapping to containers on user defined mapping. We show that it allows public access to these containers.

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

###User Defined NFS Volumes
FDCS supports user defined NFS volumes and their management.  Multiple containers can use the same volume in the same time period. This is useful if two containers need access to shared data. For example, if one container writes and the other reads the data. User defined volume data persist even when the containers that reference the volume are removed. Further since FDCS mounts user defined volumes on a NFS mount point the volumes are shared by all the host in the FDCS cluster. Thus, containers on different hosts can easily share data.

In this example we show how two users, docker_user_1 and docker_user_2, leverage user defined volumes to share and persist there data. docker_user_1's volumeA is only accessable to its containers.  docker_user_2's volumeA is only accessable to its containers. 

docker_user-1 creates volumeA. List volume shows the same volume on all hosts in the FDCS cluster. This means no matter where docker_user-1's containers are launched on the FDCS cluster they can access volumeA. 
```
$ docker --config ~/docker-user-1 volume create -d nfs --name volumeA
volumeA 
$ docker --config ~/docker-user-1 volume ls
DRIVER              VOLUME NAME
nfs               docker-host-1/volumeA
nfs               docker-host-2/volumeA
```
docker_user-1 creates two container.  One writes to the volume and the other reads the data that was just written previously to the volume. 
```

$ docker --config ~/docker-user-1 run --rm -v volumeA:/data busybox sh -c "echo docker_user_1 hello  > /data/file.txt"
$ docker --config ~/docker-user-1 run --rm -v volumeA:/data busybox sh -c "cat /data/file.txt"
docker_user_1 hello

```
docker_user-2 does has no access to the volume created by docker_user-1. 
```

$ docker --config ~/docker-user-2 volume ls
DRIVER              VOLUME NAME
$ docker --config ~/docker-user-2 run --rm -v volumeA:/data  busybox sh -c "cat /data/file.txt"
cat: can't open '/data/file.txt': No such file or directory
```
docker_user-2 creates another volume using the same name used by docker_user-1. 
```
$ docker --config ~/docker-user-2 volume create -d nfs --name volumeA
volumeA
$ docker --config ~/docker-user-2 volume ls
DRIVER              VOLUME NAME
nfs               docker-host-1/volumeA
nfs               docker-host-2/volumeA
```
docker_user-2 creates two container.  One writes to the volume and the other reads the data that was just written previously to the volume. 
```
$ docker --config ~/docker-user-2 run --rm -v volumeA:/data busybox sh -c "echo docker_user_2 hello  > /data/file.txt"
$ docker --config ~/docker-user-2 run --rm -v volumeA:/data busybox sh -c "cat /data/file.txt"
docker_user_2 hello
```
docker_user-1 again reads the data that was previously to this volume. Notice that the data on the two volumes is different.
```
$ docker --config ~/docker-user-1 run --rm -v volumeA:/data busybox sh -c "cat /data/file.txt"
docker_user_1 hello

```
docker_user-1 an d docker_user-1 remove their volumes.
```

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
###orion and mongodb
The following exampes use a service composed onf FIWARE's orion and mongodb to  demonstrate FDCS's support of user defined overlay networks, user defined NFS volumes, links and volume-from.  
####orion and mongodb: using user defined overlay network and nfs volume
The FIWARE orion and mongodb containers are connected over a user defined overlay network. Mongdb's data persists by mounting it to a user defined NFS volume.


The is creates a use defined overlay network called front.
```
$ docker network create --driver overlay front
f3cda3053dc649fd056ba9dcfe0b423f8cfd9fb8f2eaecd9ede1ccd6465d5317
docker-user-1@rcc-hrl-kvg-558:~$ docker network inspect front
[
    {
        "Name": "front",
        "Id": "f3cda3053dc649fd056ba9dcfe0b423f8cfd9fb8f2eaecd9ede1ccd6465d5317",
        "Scope": "global",
        "Driver": "overlay",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1/24"
                }
            ]
        },
        "Containers": {},
        "Options": {}
    }
]
```
The user creates a use defined NFS volume.
```
$ docker volume create -d nfs --name mongodata
mongodata
$ docker volume ls
DRIVER              VOLUME NAME
nfs                 mongodata
nfs                 mongodata

```
The user creates a mongo container, attaches it to the front network and mounts mongodata NFS volume.
```

$ docker run -d -v mongodata:/data/db --net front --name mongo mongo:3.2 --nojournal
1c487cd5236eca3a171a508aa232c749422690834fa54163d6443ef633056673
```
The user creates a FIWARE orion container and attaches it to the front network.
```
$ docker run -d -p 1026 --net front --name orion fiware/orion -dbhost mongo
a5155e4986385606863293f02dcad0432c8b4e3af413c210a4a03c2b16548557
```
The user creates inspects the front network. We observe that both containers are attached to the networks.
```
$ docker network inspect front
[
    {
        "Name": "front",
        "Id": "f3cda3053dc649fd056ba9dcfe0b423f8cfd9fb8f2eaecd9ede1ccd6465d5317",
        "Scope": "global",
        "Driver": "overlay",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1/24"
                }
            ]
        },
        "Containers": {
            "1c487cd5236eca3a171a508aa232c749422690834fa54163d6443ef633056673": {
                "Name": "mongo",
                "EndpointID": "c114672826177a26486526cc19e4d54dc7e6a326155ce4ed0b3a1ef5ac203c0b",
                "MacAddress": "02:42:0a:00:00:02",
                "IPv4Address": "10.0.0.2/24",
                "IPv6Address": ""
            },
            "a5155e4986385606863293f02dcad0432c8b4e3af413c210a4a03c2b16548557": {
                "Name": "orion",
                "EndpointID": "def63efe47844a630ff2e4c784c6b1462b65b38dcd4d648d7d907c2c9044b9fc",
                "MacAddress": "02:42:0a:00:00:03",
                "IPv4Address": "10.0.0.3/24",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]
```
The user verifies that the containers are started.
```

docker-user-1@rcc-hrl-kvg-558:~$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
a5155e498638        fiware/orion        "/usr/bin/contextBrok"   2 minutes ago       Up 2 minutes        130.206.119.32:32773->1026/tcp   docker-host-3/orion
1c487cd5236e        mongo:3.2           "/entrypoint.sh --noj"   2 minutes ago       Up 2 minutes        27017/tcp                        docker-host-2/mongo

```
The user verifies that the service is working as expected:
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
```
Initially the data base is empty.
```
$ curl ${port}/v2/entities
[]
```
The user adds and an entry to the database.
```
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
Now we demonstrate that the data on NFS volume mongodata persists by
removing orion and mongo and then launching them again.
```
docker-user-1@rcc-hrl-kvg-558:~$ docker rm -fv orion
orion
docker-user-1@rcc-hrl-kvg-558:~$ docker rm -fv mongo
mongo
docker-user-1@rcc-hrl-kvg-558:~$ docker run -d -v mongodata:/data/db --net front --name mongo mongo:3.2 --nojournal
651c807290614313956d8d72c521684745ce66b52cee9622fc9154776f4d283b
docker-user-1@rcc-hrl-kvg-558:~$ docker run -d -p 1026 --net front --name orion fiware/orion -dbhost mongo
f343271767472660c1b2c9093986470d82429a7eabd4cd17e57f7fe43afe6c78

docker-user-1@rcc-hrl-kvg-558:~$ port=$(docker port orion 1026)
docker-user-1@rcc-hrl-kvg-558:~$ curl ${port}/v2/entities
[{"id":"Room2","type":"Room","pressure":{"type":"Number","value":720.000000},"temperature":{"type":"Number","value":23.000000}}]
```


####Orion and mongo: using referencing with link and volume-from. 
In this example we use fiware's orion and mongodb to demonstrate container referencing with flags --link and --volume-from. 

It is important to note FDCS supports links and volume-from, however it is preferable to use user defined overlay networks and NFS volumes.  Docker has deprecated links.  NFS volumes are shared by all docker hosts in the FDCS cluster and thus contains referencing NFS volume may run on any host in the FDCS cluster.  Whereas containers that reference other containers with the volume-from flag must all run on the same docker host. 

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
