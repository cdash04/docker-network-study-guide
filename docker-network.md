# Docker networking

7 types of Docker network.

- [Docker networking](#docker-networking)
  - [Default Bridge](#default-bridge)
  - [User-defined bridge](#user-defined-bridge)
    - [Container to container DNS](#container-to-container-dns)
  - [Host](#host)
  - [Mac VLAN (Bridge mode)](#mac-vlan-bridge-mode)
  - [Mac VLAN (802.1Q trunk bridge mode)](#mac-vlan-8021q-trunk-bridge-mode)
  - [IPvlan (L2)](#ipvlan-l2)
  - [IPvlan (L3)](#ipvlan-l3)
  - [Overlay](#overlay)
  - [None](#none)

## Default Bridge

- docker network by default
- docker0 is the default virtual bridge (ip address is 172.17.0.1)
- The command `docker network ls` shows the list of docker network currently defined

```shell
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
32c30ab271ab   bridge    bridge    local
a205e2b4b6aa   host      host      local
694e196b09c4   none      null      local
```

The driver column makes reference to which of the 7 network type it is.

The network named `bridge` is the default bridge network of docker. Every container ran without a specified network will be part of this network. (driver = network type).

Lets run 3 docker containers, 2 busybox distro (named thor and mjolnir) and 1 nginx instance (named stormbreaker).

`$ docker run -itd --rm --name thor busybox`

`$ docker run -itd --rm --name mjolnir busybox`

`$ docker run -itd --rm --name stormbreaker nginx`

We can see our 3 docker containers running with `docker ps`

```shell
$ docker ps

CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
b3bf7f30ebbb   nginx     "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   80/tcp    stormbreaker
6612b99885a4   busybox   "sh"                     4 minutes ago   Up 4 minutes             mjolnir
7e0a1c393365   busybox   "sh"                     5 minutes ago   Up 5 minutes             thor
```

Since no network were specified when we created those 3 containers, 

It can be shown with the following command `docker network inspect <NETWORK-ID-OR-NETWORK-NAME>`. In this case, the network name is `bridge`. So th command will be `docker network inspect bridge`.

The command's output will look something like this:

```json
[
    {
        "Name": "bridge",
        "Id": "32c30ab271abe46aa7c4bccd8c1479552a5777a97a811a695ce0259be9ab3e1e",
        "Created": "2025-11-10T15:04:06.120098209Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "680c53db5e68b7b4b8b898d936faa41d6aa3cefe98a7425d1bf5b5a7448ee6cf": {
                "Name": "thor",
                "EndpointID": "27459b660170a95983a62e131a980c4e359ba24310045216d9333f4d706331a7",
                "MacAddress": "8a:81:05:13:72:49",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "72d7240efc2b4ea9077da6243d86b394a245a34faa204a38489d3240fecf017b": {
                "Name": "stormbreaker",
                "EndpointID": "96b3ac95d70727820609c2eb8f6b882102c8f23946b18007bb261118a1950d3b",
                "MacAddress": "8a:40:ff:94:64:6c",
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            },
            "bda35d7c4754f64b22423acb642a5a7acb15473ceaaea75ecbb5b82dd8288f35": {
                "Name": "mjolnir",
                "EndpointID": "aa88f777f752b13dca7acf2cf0ab3960b074f45726759d2309f36eabfcdaaef5",
                "MacAddress": "76:8b:65:b7:d1:3f",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "65535"
        },
        "Labels": {}
    }
]
```

We can see from this part of the output that the bridge subnet is `172.17.0.0/16`, meaning that every docker created without a specified docker network will have an IP address that is part of this subdomain.

```json
"IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
```

The command also shows the IP address of every container that is part of the `bridge` network.

```json
"Containers": {
            "680c53db5e68b7b4b8b898d936faa41d6aa3cefe98a7425d1bf5b5a7448ee6cf": {
                "Name": "thor",
                "EndpointID": "27459b660170a95983a62e131a980c4e359ba24310045216d9333f4d706331a7",
                "MacAddress": "8a:81:05:13:72:49",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "72d7240efc2b4ea9077da6243d86b394a245a34faa204a38489d3240fecf017b": {
                "Name": "stormbreaker",
                "EndpointID": "96b3ac95d70727820609c2eb8f6b882102c8f23946b18007bb261118a1950d3b",
                "MacAddress": "8a:40:ff:94:64:6c",
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            },
            "bda35d7c4754f64b22423acb642a5a7acb15473ceaaea75ecbb5b82dd8288f35": {
                "Name": "mjolnir",
                "EndpointID": "aa88f777f752b13dca7acf2cf0ab3960b074f45726759d2309f36eabfcdaaef5",
                "MacAddress": "76:8b:65:b7:d1:3f",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
```

The tree newly created containers had the following IP address attributed to them: `172.17.0.2/16`, `172.17.0.4/16` and `172.17.0.3/16`.

Now lets access the shell of one of our docker container. In order to that we execute the following command: `docker exec -it thor sh`.

In the shell of our container, lets look at its IP address:

```bash
$ ip add

...
11: eth0@if33: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 65535 qdisc noqueue 
    link/ether 8a:81:05:13:72:49 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

We can see it's `172.17.0.2/16`

Now lets try to ping *mjolnir* from the *thor* container:

```bash
$ ping 172.17.0.3

PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.190 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.153 ms
64 bytes from 172.17.0.3: seq=2 ttl=64 time=0.194 ms
64 bytes from 172.17.0.3: seq=3 ttl=64 time=0.185 ms
64 bytes from 172.17.0.3: seq=4 ttl=64 time=0.181 ms
^C
--- 172.17.0.3 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.153/0.180/0.194 ms
```
So on bridge network, containers can access each other automatically and also access the internet.

Now lets try to reach my nginx container outside of the bridge network.

```
$ curl -v localhost:80                          
* Host localhost:80 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:80...
* connect to ::1 port 80 from ::1 port 56830 failed: Connection refused
*   Trying 127.0.0.1:80...
* connect to 127.0.0.1 port 80 from 127.0.0.1 port 56831 failed: Connection refused
* Failed to connect to localhost port 80 after 0 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to localhost port 80 after 0 ms: Couldn't connect to server
```

By default, every machine outside the bridge network can't access it. To do so we need to map the port we want to reach manually when deploying 

So now redeploy our nginx container with its port 80 mapped to our port 80:

```bash
$ docker stop stormbreaker
$ docker run -itd --rm --name stormbreaker -p 80:80 nginx
```

With `docker ps` we can not see that the port `80` of our nginx container has been mapped to the port 80 of our container.

And now we can access our nginx container:

```bash
$ curl -v localhost:80
* Host localhost:80 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:80...
* Connected to localhost (::1) port 80
...
```
## User-defined bridge

Like the default bridge but created and setup by us.

Lets create a docker bridge network:

`$ docker network create asgard`

We can now see out bridge network with the command `docker network ls`

```bash
$ docker network ls                  
NETWORK ID     NAME      DRIVER    SCOPE
72fcd743e9c2   asgard    bridge    local
32c30ab271ab   bridge    bridge    local
a205e2b4b6aa   host      host      local
694e196b09c4   none      null      local
```

And we can get its network IP address with `$ docker network inspect asgard`

```json
$ docker network inspect asgard
[
    {
        "Name": "asgard",
        "Id": "72fcd743e9c2627d4344a645772a0fbfc2039afa847851a5a4e1038d8246b855",
        "Created": "2025-11-14T18:22:02.5107058Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.enable_ipv4": "true",
            "com.docker.network.enable_ipv6": "false"
        },
        "Labels": {}
    }
]
```
We can see the network of asgard is `172.18.0.0/16`.

Now lets add containers to our new network. It's basically the same as before but now we add the `--network` argument to specify which network the container will be a part off.

Lets add two *busybox* 

`$ docker run -itd --rm --network asgard --name loki busybox`

`$ docker run -itd --rm --network asgard --name odin busybox`

Now when we inspect the network `asgard` once again we can expect to see our two newly created container with IP address that are without the sub network `172.18.0.0/16`.

```json
$ docker network inspect asgard                            
[
...
        "Containers": {
            "bd7dd57cd194f0070bd42a295fd3903a9ac349ef45ebc0d8afec81390da90f2c": {
                "Name": "loki",
                "EndpointID": "844ba4e45d42258b0bf913b36a7098287950b82358554649d6d6071058e48295",
                "MacAddress": "52:ae:08:af:9a:c9",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "e4c02447360863bc8725a56e17ad755b2721c32ef29469f68cc5e334d5d19747": {
                "Name": "odin",
                "EndpointID": "571d5320bd5d9623c2404d99275a54b0f0b297ee443fcc774be0898f9a093d3c",
                "MacAddress": "9a:6b:22:cc:32:8a",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.enable_ipv4": "true",
            "com.docker.network.enable_ipv6": "false"
        },
        "Labels": {}
    }
]
```

User-defined bridge network are preferred over the default bridge network because it's an additional security layer around the containers. For example, lets try to connect to a container in the network `asgard`, lets say `loki` and ping a container in the default bridge like `thor`.

```bash
$ docker exec -it loki sh
$ ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
--- 172.17.0.2 ping statistics ---
11 packets transmitted, 0 packets received, 100% packet loss
```

As we can see, containers in the bridge network `asgard` can't access the containers in the default bridge.

### Container to container DNS

Inside a user-defined bridge network, you can access other containers with only the container name. For example, from the container named `loki`, we can communicate with the container named `odin` simply using its name:

```
$ ping odin
PING odin (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.145 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.113 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.196 ms
^C
--- odin ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.113/0.151/0.196 ms
```

## Host

- Simplest type.
- When a container is specified the host network, the container will use the host (duh).
- No ports need to be specified since it will uses the hosts ports.
- Can be useful for application like VPN where you want your application directly connected to your host and not isolated inside a bridge.
- Downside is there's no isolation by default so there can be security risks associated with this network solution.

In order to run a container in a host network, simply add `--network host` to your command to start a docker conainer:

`$ docker run -itd --rm --network host --name odin busybox`

## Mac VLAN (Bridge mode)

- Mac VLAN network connect containers directly to the host *network*.
- Each container will get their own IP address and even MAC address from the host network.

Now lets create a Mac VLAN network with the same `docker network create` command as before except we will need to specify which driver to use with the `-d` parameter.

When creating a Mac VLAN network we also need to specify:
- The *subnet* of our local network
- The *gateway* (the local IP address of the router).
- And the parent interface (*en0* in my case).

```
$ docker network create -d macvlan \
    --subnet 192.168.18.0/24 \
    --gateway 192.168.18.1 \
    -o parent=eth0 \
    newasgard
```

## Mac VLAN (802.1Q trunk bridge mode)

## IPvlan (L2)

On `Mac VLAN` network, each containers created within the network were given a MAC address. Multiple MAC address on a given network interface will not work if the interface doesn't have a promiscuous mode on. Thus, it may not work out of the box on certain network.

However, with IPvlan, you get the cool benefice of Mac VLAN like your containers being connected directly to your network, but it allows the different containers to share the same MAC address.

Lets create a IPvlan network:

- ipvlan by default will be L2, so you just need to specify `-d ipvlan`

```bash
$ docker network create -d ipvlan \
    --subnet 10.10.10.0/24 \
    --gateway 10.10.10.1 \
    -o parent=eth0 \
    newasgard
```

## IPvlan (L3)

The last docker networks have heavily focussed on layer 2 so far (ARP, MAC, Switch). But with L3, we focus on layer 3, so IP adresses.

So, in a nutshell, our containers wont be connected on an interface, but instead a router. So there won't be broadcast between the containers within the network.

Lets create our IPvlan L3 network:

- it starts just like an IPvlan L2 with the `-d ipvlan`
- However, the subnet created are completely new and out of the host network
- No gateway are provided since the gateway will be the specified parent interface

```bash
$ docker network create -d ipvlan \
    --subnet 192.168.94.0/24 \
    -o parent=eth0 -o ipvlan_mode=l3 \
    --subnet 192.168.95.0/24 \
    newasgard
```

Now lets add containers to our network:

- We have to specify an IP adresse to make it knows which sub network the containers will be a part of

```bash
$ docker run -itd --rm --network newasgard \
    --ip 192.168.94.7 \
    -name thor busybox
```
```bash
$ docker run -itd --rm --network newasgard \
    --ip 192.168.94.8 \
    -name mjolnir busybox
```
You can see that our four containers are inside the network with `$ docker network inspect newasgard`

Now lets access thor's container's shell and ping its friends:

- You can ping containers in your own subnet
- You can also ping containers in other subnet

```bash
$ docker exec -it thor sh
$ ping mjolnir 
$ ping loki
```
IPvlan L3 turns your host into a router, allowing you to run router container networks that are routed. 

## Overlay

Used for orchestrator like Docker swarm.

## None

The most secured network, because there is no network.

It already exist and can be seen when listing your current docker's network:

```bash
$ docker network ls
...
694e196b09c4   none        null      local
```
