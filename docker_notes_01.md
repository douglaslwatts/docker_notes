# Getting Started with Docker

---

`docker run -dti debian`

-d => run in detached mode in the background <br>
-t => container will have a psuedo tty open <br>
-i => run in interactive mode, such that stdin is open for inputs <br>

so, -ti is what makes it possible to open a shell in the running container while
the -d keeps the container running in the background so it does not immediately shutdown after starting.

We can think of it as telling Docker to run a detached, terminal interactive container.

`docker ps` => view running containers

`docker --help` <br>
`docker --help images` <br>
`docker --help images prune` <br>

## stop a running container

`docker stop <container_id | container name>`

## see json output for container data

`docker inspect <container_id | container name>`

## download image ...default tag is "latest" (`docker run` will pull an image if needed)

`docker pull nginx`

## see the images which exist on this system

`docker images`

## show image layers line by line

`docker history`

## do not truncate the sha256 hash of the images in the output

`docker images --no-trunc`

## tag the image with a custom tag name

`docker tag nginx:latest nginx:my_stable_nginx`

## -t specifies a tag for the image built, the . says use Dockerfile in pwd

`docker build -t mynginx .`

## run the container just built from the Dockerfile

`docker run mynginx`

## remove the image with that tag ...sometimes -f may be needed

`docker rmi nginx:my_stable_nginx`

## interactively remove dangling images (images not associated with a container)

`docker image prune`

## remove all stopped containers, unused networks, dangling images, and build cache

`docker system prune -a`

## see how much disk space docker is using

`docker system df`

storage => /var/lib/docker
