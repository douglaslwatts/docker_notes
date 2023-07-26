# Docker Notes 04

## Container Networking

---

When we expose a port in a Docker container, a path is created from the outside of the container.
That path goes through the networking layers, into a virtual network, and into that container.
Other containers can connect to that container by going out to the host, turning around, and going
in along that path. Docker offers more efficient ways to set up networking between containers.

We can see the default set of virtual networks set up by Docker:

```
> docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
009faec6138b   bridge    bridge    local
fc241c1ccb21   host      host      local
fd0e197e8979   none      null      local
```

1. `bridge` is used by containers that do not specify any network to be placed in.
2. `host` is used by containers that do not have any networking isolation, which comes with security
   concerns.
3. `none` is used when a container should have no networking at all.

We can create a network of our own:

`docker network create animals`

```
> docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
00f9ea6c38b1   animals   bridge    local
009faec6138b   bridge    bridge    local
fc241c1ccb21   host      host      local
fd0e197e8979   none      null      local
```

Now lets start two containers on that network in separate terminals.

First terminal:

`docker run --rm -ti --net animals --name dog-server ubuntu:14.04 bash`

Second terminal:

`docker run --rm -ti --net animals --name cat-server ubuntu:14.04 bash`

from each server ping the other:

First terminal: `ping -c 3 cat-server`

Second terminal: `ping -c 3 dog-server`

Now, we can use `netcat` to further show this:

First terminal:

`nc -lp 45678`

Second terminal:

`nc dog-server 45678`

Now from either side we can send to the other:

First terminal:

`Woof Woof!`

Second terminal:

`Meoooow!`

We see that the message is sent to the other server in both directions.
