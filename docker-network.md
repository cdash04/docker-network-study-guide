# Docker networking

7 types of Docker network.

## Default Bridge

- docker network by default
- docker0 is the default virtual bridge (ip address is 172.17.0.1)
- The command `docker network ls` shows the list of docker network currently defined

```shell
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
1e0649fa54f0   bridge    bridge    local
28bcf78d8910   host      host      local
5149a6bf356b   none      null      local
```

The driver column makes reference to which of the 7 network type it is.

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

For those 3 containers, docker automatically created 3 virtual eth interface
