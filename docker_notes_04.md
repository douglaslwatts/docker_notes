# Docker Notes 04

## Container Networking

---

When we expose a port in a Docker container, a path is created from the outside of the container.
That path goes through the networking layers, into a virtual network, and into that container.
Other containers can connect to that container by going out to the host, turning around, and going
in along that path. Docker offers more efficient ways to set up networking between containers.

We can see the default set of virtual networks set up by Docker:

<pre>
dlwatts@terra ~ > docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
009faec6138b   bridge    bridge    local
fc241c1ccb21   host      host      local
fd0e197e8979   none      null      local
dlwatts@terra ~ >
</pre>

1. `bridge` is used by containers that do not specify any network to be placed in.
2. `host` is used by containers that do not have any networking isolation, which comes with security
   concerns.
3. `none` is used when a container should have no networking at all.

We can create a network of our own:

`docker network create animals`

<pre>
dlwatts@terra ~ > docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
00f9ea6c38b1   animals   bridge    local
009faec6138b   bridge    bridge    local
fc241c1ccb21   host      host      local
fd0e197e8979   none      null      local
dlwatts@terra ~ >
</pre>

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

We see that the message is sent to the other server in both directions. A server can also be on
multiple networks. Let's add another network and add the cat-server to it, but not the dog-server.

`docker network create cats-only`

`docker network connect cats-only cat-server`

Now let's start a new server in another terminal called jaguar-server.

`docker run --rm -ti --net cats-only --name jaguar-server ubuntu:14.04 bash`

Now, we can `ping` the jaguar-server and the dog-server from the cat-server because it is on both
the animals network and the cats-only network. However, the dog-server and the jaguar-server can only
`ping` the cat-server because the jaguar-server is only on the cats-only network, and the dog-server
is only on the animals network.

### Legacy Linking

---

- Links all ports from a server, but it is unidirectional
- Environment variable are shared, but it is unidirectional
- The startup order is important as there must be a running server to link to, which makes
  orchestration more challenging
- If carefully done, restarts can not break the links

Generally, it is best to stick with more modern networking options offered by Docker unless there
is a specific need of the features offered by legacy linking.

For an example we can start a server with an environment variable to share, and start another server
that links to it.

First terminal:

`docker run --rm -ti -e OPENAI_API_KEY=lksdjflskjdf --name labrador-server archlinux bash`

Second terminal:

`docker run --rm -ti --link labrador-server --name bernese-server archlinux bash`

Now, if we run the `env` in the bernese-server, we can see the environment variable we set for the
labrador-server as `LABRADOR_SERVER_ENV_OPENAI_API_KEY`, however we do not see any linked servers'
environment variables in the labrador-server.

Another thing to note is that we can `ping` the labrador-server from the bernese-server, but not the
other way around.
