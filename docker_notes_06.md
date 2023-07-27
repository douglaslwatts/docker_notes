# Docker Notes 06

## Volumes

---

Volumes are virtual discs to store and share data between containers, between containers and the host,
or both. There are two types of volumes:

1. Persistent: These volumes persist after the container is stopped or even deleted
2. Ephemeral: These volumes only exist as long as they are being used, so they no longer exist once
   the container using them is stopped.

A volume is not a part of the image. No volumes are included if you download or upload an image. they
are only for data local to the host.

- Share a directory with the host

  - We **must** specify the absolute path of the host directory to share for the data so persist
  - We specify the absolute path where we want the directory to live in the container

  `docker run -ti -v /home/dlwatts/example_dir:/example_dir archlinux bash`

  <pre>
  dlwatts@terra ~ > docker run -ti -v /home/dlwatts/example_dir:/example_dir archlinux bash
  [root@464097c80b8c /]# ls -d example_dir/
  example_dir/
  [root@464097c80b8c /]# echo 'Stay positive, always keep learning, be kind, be helpful, share knowledge' > example_dir/note
  [root@464097c80b8c /]# cat example_dir/note
  Stay positive, always keep learning, be kind, be helpful, share knowledge
  [root@464097c80b8c /]# exit
  exit
  dlwatts@terra ~ > cat example_dir/note
  Stay positive, always keep learning, be kind, be helpful, share knowledge
  dlwatts@terra ~ >
  </pre>

  We can see that once we exited the container, the data persisted in the shared
  directory.

- Share a file with a container

  - Sharing a file works exactly the same way as sharing a directory
  - Make sure the file exists on the host before sharing it or Docker will share it as a file

    <pre>
    dlwatts@terra ~ > echo 'The sun is shining' > sunny
    dlwatts@terra ~ > cat sunny
    The sun is shining
    dlwatts@terra ~ > docker run -ti -v /home/dlwatts/sunny:/sunny archlinux bash
    [root@dea450824a66 /]# ls sunny
    sunny
    [root@dea450824a66 /]# cat sunny
    The sun is shining
    [root@dea450824a66 /]# echo 'It is a beautiful day' >> sunny
    [root@dea450824a66 /]# cat sunny
    The sun is shining
    It is a beautiful day
    [root@dea450824a66 /]# exit
    exit
    dlwatts@terra ~ > cat sunny
    The sun is shining
    It is a beautiful day
    dlwatts@terra ~ >
    </pre>

## Sharing Data Between containers

---

We can also share data between containers that will not be shared with the host using the
`--volumes-from` option.

First terminal:

<pre>
dlwatts@terra ~ > docker run -ti -v /shared-data archlinux bash
[root@661373d02621 /]# ls shared-data/
[root@661373d02621 /]# echo -n Hello > shared-data/hello_docker
[root@661373d02621 /]# cat shared-data/hello_docker
Hello[root@661373d02621 /]#
</pre>

Second Terminal:

<pre>
dlwatts@terra ~ > docker_ps -l
CONTAINER ID    661373d02621
IMAGE           archlinux
CREATED         38 seconds ago
STATUS          Up 37 seconds
PORTS
NAMES           competent_wescoff
dlwatts@terra ~ > docker run -ti --volumes-from competent_wescoff archlinux bash
[root@94e7e3ce9698 /]# cat shared-data/hello_docker
Hello[root@94e7e3ce9698 /]# echo ' Docker' >> shared-data/hello_docker
[root@94e7e3ce9698 /]# cat shared-data/hello_docker
Hello Docker
[root@94e7e3ce9698 /]#
</pre>

If we exit the original container that created the volume, the directory persists on the other
container and we can start another container that uses it as well.

Back in first terminal:

<pre>
Hello[root@661373d02621 /]# exit
dlwatts@terra ~ > docker run -ti --volumes-from competent_wescoff archlinux bash
[root@94e7e3ce9698 /]# cat shared-data/hello_docker
Hello Docker
[root@94e7e3ce9698 /]# exit
dlwatts@terra ~ >
</pre>

Back in second terminal:

<pre>
[root@94e7e3ce9698 /]# exit
exit
dlwatts@terra ~ > ls shared-data
ls: cannot access 'shared-data': No such file or directory
dlwatts@terra ~ >
</pre>

We can see that the directory is shared only between the containers and does not exist on the host.
