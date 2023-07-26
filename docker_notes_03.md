# Docker Notes 03

## Managing Containers

---

**`docker logs <containerName|ID>`**

`docker run --name example -d ubuntu bash -c "catt /etc/passwd"`

Since we misspelled the command, the container will crash and we can see the logs from that as below.

`docker logs example`

**`docker kill <containerName|ID>`**

If we have a long running container as below:

`docker run -d --name forever_container ubuntu bash -c "while [ true ]; do sleep 1; done"`

Then, we can kill it as below:

`docker kill forever_container`

and then remove it:

`docker rm forever_container`

When using named containers, we sometimes want to reuse a name and must rm the old one with that
name first.

**`--memory`**

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


