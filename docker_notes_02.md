# Docker Notes 02

## Running Containers

---

We can run as many containers based on a given image as we want

## Run a container from an image, but give it a custom name

`docker run -dti --name=custom_name debian`

## Remove a stopped container. `rmi` removes images, while `rm` removes containers. <br>

`docker stop custom_name`

`docker rm custom_name`

## A container can also be run without detaching it

---

`docker pull ubuntu:latest` <-- latest is actually the default

`docker run -ti ubuntu bash`

Now we are in a running ubuntu container and can press CTRL-D or type exit to leave it.

## Commit our changes to a container to create our own image from it

---

Now that we have exited the ubuntu container above, lets get its ID or NAME and commit that

`docker ps -l` <-- see the last container exited

Let's say the container ID was a1e89f29a77b and its NAME was eloquent_franklin. Then we can use
either the first 4 or 5 characters of the ID or the NAME in the below command. By doing this we
commit that container as an image with the given TAG.

`docker commit a1e89 my_first_commit_image`

What is really happening behind the scenes
is that the container is committed as an image with a given sha256 and then that image is given
the TAG we provide. So the above command is a shortcut for the below two commands.

`docker commit a1389` <-- outputs the sha256 of the new image created

`docker tag <sha256 from commit command above> <tag/name for the image>`

So, now there is an image from that container named my_first_commit_image which has the state of
that container including any files created or changed within it. Let's run it.

`docker run -ti my_first_commit_image`

## docker run

---

When we run a container there are a few things to remember:

- Containers have one main process
- The container stops when its main process exits
  (even if we have started other processes in it that have not exited)
- Containers have names (if we do not give it one, docker will make one up)

### `--rm`

We can use the `--rm` flag to tell docker not to keep the container once exited. This can keep our
system from saving so many stopped containers and is a shortcut for running `docker rm containerName`
after the container has exited.

`docker ps -a` <-- list all containers, including stopped ones

`docker run --rm -ti ubuntu` <-- once this container is exited it is automatically removed

### **running specific commands in a container**

We can provide a specific command or set of commands for the container to run when we run it.

`docker run --rm -ti ubuntu bash -c "echo Hello Docker; sleep 3s; echo All Done"`

The above container will run the commands provided via bash and then exit. We can also run just one
command as below.

`docker run --rm -ti ubuntu bash -c "echo Hello Docker"`

or

`docker run --rm -ti ubuntu bash -c "echo Hello Docker && sleep 3s && echo All Done"`

## docker attach

---

We can attach to a running container by its ID or NAME.

`docker run --rm -dti ubuntu`

`docker attach <NAME|ID>`

We can leave the container and have it continue running by typing CTRL-p, CTRL-q and then reattach
later if we need.

We can add more processes to a running container. Below, we run a container in attached mode, then
run Bash from another terminal. Once the initial one exits, however so does the second.

`docker run --rm -ti ubuntu bash`

now from another terminal

`docker ps` <-- to see running container's NAME

`docker exec -ti <container name> bash`

Now we are in the same container from 2 different terminals. when the container is stopped, both
terminals will be detached from it. The container could be stopped from the first terminal which
started it in attached mode or from a third terminal via `docker stop <container>`.
