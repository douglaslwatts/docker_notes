# Docker Notes 05

## Images

---

List images downloaded on this machine

<pre>
dlwatts@terra ~ > docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
archlinux     latest    1105a6ef0052   2 days ago      432MB
fedora        latest    ad2032316c26   3 weeks ago     190MB
nginx         latest    021283c8eb95   3 weeks ago     187MB
debian        latest    3676c78a12ad   3 weeks ago     116MB
ubuntu        latest    5a81c4b8502e   4 weeks ago     77.8MB
hello-world   latest    feb5d9fea6a5   22 months ago   13.3kB
ubuntu        14.04     13b66b487594   2 years ago     197MB
dlwatts@terra ~ >
</pre>

It is completely unnecessary, but we can define a format for outputting docker commands with the
`--format` option. I have set up an environment variable and an alias to make the output of the
`docker ps` command a little nicer. This can be set up wherever your environment variables and
aliases are defined.

<pre>
dlwatts@terra ~ > echo $DOCKER_PS_FORMAT
CONTAINER ID    {{.ID}}
IMAGE           {{.Image}}
CREATED         {{.RunningFor}}
STATUS          {{.Status}}
PORTS           {{.Ports}}
NAMES           {{.Names}}
dlwatts@terra ~ > grep PS_ .zenv_vars
export DOCKER_PS_FORMAT='CONTAINER ID\t{{.ID}}\nIMAGE\t\t{{.Image}}\nCREATED\t\t{{.RunningFor}}\nSTATUS\t\t{{.Status}}\nPORTS\t\t{{.Ports}}\nNAMES\t\t{{.Names}}'
dlwatts@terra ~ > grep docker_ps .zsh_aliases
alias docker_ps='docker ps --format=$DOCKER_PS_FORMAT'
dlwatts@terra ~ >
</pre>

Tagging images gives them names and this can be done by committing a stopped container. Checking the last
container stopped we see:

<pre>
dlwatts@terra ~ > docker_ps -l
CONTAINER ID    50f4e5dbbd6d
IMAGE           archlinux
CREATED         4 minutes ago
STATUS          Exited (127) 17 minutes ago
PORTS
NAMES           strange_cori
dlwatts@terra ~ >
</pre>

Let's commit that container with no tag specified.

`docker commit 50f4e5dbbd6d archlinux-server`

<pre>
dlwatts@terra ~ > docker images
REPOSITORY         TAG       IMAGE ID       CREATED         SIZE
archlinux-server   latest    6465d1128073   3 seconds ago   432MB
archlinux          latest    1105a6ef0052   2 days ago      432MB
fedora             latest    ad2032316c26   3 weeks ago     190MB
nginx              latest    021283c8eb95   3 weeks ago     187MB
debian             latest    3676c78a12ad   3 weeks ago     116MB
ubuntu             latest    5a81c4b8502e   4 weeks ago     77.8MB
hello-world        latest    feb5d9fea6a5   22 months ago   13.3kB
ubuntu             14.04     13b66b487594   2 years ago     197MB
dlwatts@terra ~ >
</pre>

We can see that a default tag "latest" is given when no tag is specified.

Now let's save another version with a custom tag.

`docker commit 50f4e5dbbd6d archlinux-server:v.1.0`

<pre>
dlwatts@terra ~ > docker images
REPOSITORY         TAG       IMAGE ID       CREATED         SIZE
archlinux-server   v.1.0     c0a0f429a553   3 seconds ago   432MB
archlinux-server   latest    6465d1128073   4 minutes ago   432MB
archlinux          latest    1105a6ef0052   2 days ago      432MB
fedora             latest    ad2032316c26   3 weeks ago     190MB
nginx              latest    021283c8eb95   3 weeks ago     187MB
debian             latest    3676c78a12ad   3 weeks ago     116MB
ubuntu             latest    5a81c4b8502e   4 weeks ago     77.8MB
hello-world        latest    feb5d9fea6a5   22 months ago   13.3kB
ubuntu             14.04     13b66b487594   2 years ago     197MB
dlwatts@terra ~ >
</pre>

We can see that we now have a latest and a v.1.0. So if we run the archlinux-server without
specifying a tag, we get latest and if we specify v.1.0, we get that one.

The full naming structure is:

`registry.server.com:port/organization-name/image-name:version-tag`

Leave off the parts that are not needed for a given situation. Images can build up quickly and
use up a lot of space. Refer back to [docker_notes_01.md](https://github.com/douglaslwatts/docker_notes/blob/main/docker_notes_01.md)
for removing images which are no longer needed. If there are a lot of images, it may be a good idea
to have a shell script to handle this job.
