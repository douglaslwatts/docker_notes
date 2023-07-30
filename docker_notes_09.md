# Docker Notes 09

## Multistage builds in Docker files

---

We can limit the size of an image by using multistage builds. We can have multiple `FROM` statements
and each one can use a different base image if we want. Each `FROM` starts a new stage of the build.
Subsequent stages can copy artifacts from previous stages' images leaving behind anything such as
packages that were only needed for a short time in the build process. This means that the final
stage creates as small an image as possible.

For a simple example, create this dockerfile.

### Dockerfile

---

<pre>
FROM fedora

RUN dnf upgrade -y
RUN dnf install curl -y
RUN curl https://github.com  | grep -o git | wc -l > git_count

ENTRYPOINT echo -n "The word git appears "; echo "$(cat git_count) times in the github html"
</pre>

Now build an image from it.

<pre>
dlwatts@terra ~ > docker build -t git-count-lg .
[+] Building 7.5s (9/9) FINISHED                                                           docker:default
 => [internal] load .dockerignore                                                                    0.1s
 => => transferring context: 2B                                                                      0.0s
 => [internal] load build definition from Dockerfile                                                 0.1s
 => => transferring dockerfile: 379B                                                                 0.0s
 => [internal] load metadata for docker.io/library/fedora:latest                                     6.4s
 => [auth] library/fedora:pull token for registry-1.docker.io                                        0.0s
 => [1/4] FROM docker.io/library/fedora@sha256:8c27ac4634ce7a761728e97985ff03fa422ccdc58c5d5d38a282  0.1s
 => => resolve docker.io/library/fedora@sha256:8c27ac4634ce7a761728e97985ff03fa422ccdc58c5d5d38a282  0.1s
 => CACHED [2/4] RUN dnf upgrade -y                                                                  0.0s
 => CACHED [3/4] RUN dnf install curl -y                                                             0.0s
 => CACHED [4/4] RUN curl https://github.com  | grep -o git | wc -l > git_count                      0.0s
 => exporting to image                                                                               0.7s
 => => exporting layers                                                                              0.7s
 => => writing image sha256:454005ee3fd176f4f7c39a52288c142533ce8f6e00b29be5a115fe6f1e095b10         0.0s
 => => naming to docker.io/library/git-count-lg                                                      0.0s
dlwatts@terra ~ >
</pre>

Now break the file into two stages.

### Dockerfile

---

<pre>
FROM fedora AS builder

RUN dnf upgrade -y
RUN dnf install curl -y
RUN curl https://github.com  | grep -o git | wc -l > git_count

FROM fedora

COPY --from=builder /git_count /git_count
ENTRYPOINT echo -n "The word git appears "; echo "$(cat git_count) times in the github html"
</pre>

Now build an image from that.

<pre>
dlwatts@terra ~ > docker build -t git-count-sm .
[+] Building 0.9s (9/9) FINISHED                                                           docker:default
 => [internal] load build definition from Dockerfile                                                 0.1s
 => => transferring dockerfile: 377B                                                                 0.0s
 => [internal] load .dockerignore                                                                    0.1s
 => => transferring context: 2B                                                                      0.0s
 => [internal] load metadata for docker.io/library/fedora:latest                                     0.5s
 => [builder 1/4] FROM docker.io/library/fedora@sha256:8c27ac4634ce7a761728e97985ff03fa422ccdc58c5d  0.1s
 => => resolve docker.io/library/fedora@sha256:8c27ac4634ce7a761728e97985ff03fa422ccdc58c5d5d38a282  0.1s
 => CACHED [builder 2/4] RUN dnf upgrade -y                                                          0.0s
 => CACHED [builder 3/4] RUN dnf install curl -y                                                     0.0s
 => CACHED [builder 4/4] RUN curl https://github.com  | grep -o git | wc -l > git_count              0.0s
 => CACHED [stage-1 2/2] COPY --from=builder /git_count /git_count                                   0.0s
 => exporting to image                                                                               0.0s
 => => exporting layers                                                                              0.0s
 => => writing image sha256:f61001be89c1c78b430cbcd9cc4061755fd5450c3c89669a5dc942e0709e23df         0.0s
 => => naming to docker.io/library/git-count-sm                                                      0.0s
dlwatts@terra ~ >
</pre>

Now look at the size difference in the two resulting images and notice that they do the exact same job.

<pre>
dlwatts@terra ~ > docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
git-count-sm   latest    f61001be89c1   10 minutes ago   190MB
git-count-lg   latest    454005ee3fd1   10 minutes ago   480MB
dlwatts@terra ~ > docker run git-count-lg
The word git appears 585 times in the github html
dlwatts@terra ~ > docker run git-count-sm
The word git appears 585 times in the github html
dlwatts@terra ~ >
</pre>
