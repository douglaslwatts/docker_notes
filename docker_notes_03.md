# Docker Notes 03

## Managing Containers

---

### `docker logs <containerName|ID>`

`docker run --name example -d ubuntu bash -c "catt /etc/passwd"`

Since we misspelled the command, the container will crash and we can see the logs from that as below.

`docker logs example`

### `docker kill <containerName|ID>`

If we have a long running container as below:

`docker run -d --name forever_container ubuntu bash -c "while [ true ]; do sleep 1; done"`

Then, we can kill it as below:

`docker kill forever_container`

and then remove it:

`docker rm forever_container`

When using named containers, we sometimes want to reuse a name and must rm the old one with that
name first.

### `--memory`

We can set the maximum allowed memory for a container as below:

`docker run --memory <max-memory-allowed> <image-name> [command]`

We can limit cpu time:

`docker run --cpu-shares <int>` <-- limit cpu period relative to other containers

`docker run --cpu-quota <int>` <-- limit cpu quota in general (ex. 10% of total cpu time ever)

Orchestration, which will be later in these notes, generally requires specifying these types of limits.

## Lessons/Advice from course instructor

---

1. Do not let containers fetch dependencies when they start because libraries can be removed
   causing the containers to crash. Instead, have the containers include the dependencies inside
   the containers themselves.

2. Do not leave important data in unnamed stopped containers. The containers will be deleted
   inadvertently at some point during a clean up.

   My own take from number 2; Maybe better to have a volume such that the data would persist after
   deleting the container.

## Exposing Ports

---

- Programs in containers are isolated form the internet by default.
- Containers can be grouped into "private" networks
- Who can connect to whom is explicitly chosen
- Expose ports to let connections in
  - Explicitly specify the port inside and outside the container
  - Expose as many ports as are needed
  - Requires coordination between Containers
  - Makes it easy to find the exposed ports
- Private networks to connect between containers

#### Example:

1. Open 3 terminals
2. In the first terminal

   - Run a terminal interactive container exposing ports 45678 and 45679 exposing the same port
     on the inside as out, and name the contaner echo-server. We use ubuntu:14.04 since it has
     `nc` installed already.

     `docker run --rm -ti -p 45678:45678 -p 45679:45679 --name echo-server ubuntu:14.04 bash`

   - Inside the container run:

     `nc -lp 45678 | nc -lp 45679`

3. In the second terminal we could run `nc localhost 45678`, but lets use a docker container instead

   `docker run --rm -ti ubuntu:14.04 bash`

   - Docker provides the special name `host.docker.internal` to access localhost since the containers
     are isolated. Inside the container run:

     `nc host.docker.internal 45678`

4. In the third terminal lets do the same with port 45679

   `docker run --rm -ti ubuntu:14.04 bash`

   - Inside the container run:

     `nc host.docker.internal 45679`

5. Any input to port 45678 on the echo-server will be sent out on port 45679. So, back in the
   second terminal, in the container with `netcat` running on localhost to port 45678, type
   something and press enter.

6. Notice that whatever was typed is echoed to the third terminal since that container has
   `netcat` running on localhost to port 45679 where the echo-server sent what we input to
   port 45678.

If we do not specify the port to expose outside the container docker will choose for us.

Repeat the above example without specifying the outside port:

1. First Terminal:

   `docker run --rm -ti -p 45678 -p 45679 --name echo-server ubuntu:14.04 bash`

2. We can find what ports docker chose for outside the container in the second terminal

<pre>
dlwatts@terra ~ > docker port echo-server
45678/tcp -> 0.0.0.0:50508
45679/tcp -> 0.0.0.0:50509
dlwatts@terra ~ >
</pre>

So, lets run `netcat` locally this time

`nc localhost 50508`

3. In the third terminal lets receive the output from the echo server

   `nc localhost 50509`

4. We can get the same result as before by entering some text in the second terminal.
