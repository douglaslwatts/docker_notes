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

The following creates an example that runs an angular app using the nginx image as a base.

1. Create an Angular app

   `ng new somesite.com`

2. Create the below Docker file in the base directory of the Angular app.

    <pre>FROM node AS builder

    WORKDIR /usr/local/app
    COPY ./ /usr/local/app
    RUN npm install
    RUN npm run build

    FROM nginx
    COPY --from=builder /usr/local/app/dist/somesite.com /usr/share/nginx/html
    EXPOSE 80
    </pre>

3. Build The image, Run a container from it, and go to localhost:8080 in a browser to see the running app.

    <pre>
    dlwatts@terra ~ > cd somesite.com/
    dlwatts@terra somesite.com main* > docker build -t somesite.com .
    [+] Building 28.6s (15/15) FINISHED                                                                                                                                                                   docker:default
     => [internal] load build definition from Dockerfile                                                                                                                                                            0.1s
     => => transferring dockerfile: 358B                                                                                                                                                                            0.0s
     => [internal] load .dockerignore                                                                                                                                                                               0.1s
     => => transferring context: 2B                                                                                                                                                                                 0.0s
     => [internal] load metadata for docker.io/library/nginx:latest                                                                                                                                                 5.6s
     => [internal] load metadata for docker.io/library/node:latest                                                                                                                                                  5.6s
     => [auth] library/node:pull token for registry-1.docker.io                                                                                                                                                     0.0s
     => [auth] library/nginx:pull token for registry-1.docker.io                                                                                                                                                    0.0s
     => [builder 1/5] FROM docker.io/library/node@sha256:64b71834718b859ea389790ae56e5f2f8fa9456bf3821ff75fa28a87a09cbc09                                                                                           0.0s
     => [stage-1 1/2] FROM docker.io/library/nginx@sha256:67f9a4f10d147a6e04629340e6493c9703300ca23a2f7f3aa56fe615d75d31ca                                                                                          0.0s
     => [internal] load build context                                                                                                                                                                               0.8s
     => => transferring context: 4.89MB                                                                                                                                                                             0.7s
     => CACHED [builder 2/5] WORKDIR /usr/local/app                                                                                                                                                                 0.0s
     => [builder 3/5] COPY ./ /usr/local/app                                                                                                                                                                        8.5s
     => [builder 4/5] RUN npm install                                                                                                                                                                               3.5s
     => [builder 5/5] RUN npm run build                                                                                                                                                                             9.7s
     => CACHED [stage-1 2/2] COPY --from=builder /usr/local/app/dist/somesite.com /usr/share/nginx/html                                                                                                        0.0s
     => exporting to image                                                                                                                                                                                          0.0s
     => => exporting layers                                                                                                                                                                                         0.0s
     => => writing image sha256:e89200ba1a10b4014aede691ec441f3943153ced12614275187c2b28789e77e9                                                                                                                    0.0s
     => => naming to docker.io/library/somesite.com
    dlwatts@terra ~ somesite.com main* > docker run -d -p 8080:80 --name somesite_v.1.0 somesite.com
    ea47f24d2abeb969550ee2c82172199e41bfa52dc020dfa1f5edfc2cb4a9aadc
    dlwatts@terra ~ > docker_ps -l
    CONTAINER ID    ea47f24d2abe
    IMAGE           somesite.com
    CREATED         About a minute ago
    STATUS          Up About a minute
    PORTS           0.0.0.0:8080->80/tcp, :::8080->80/tcp
    NAMES           somesite_v.1.0
    dlwatts@terra somesite.com main* >
    </pre>

The app is now being served at localhost:8080 until the container is stopped.

Open `src/app/app.component.html` and add an `h1` just above the `h2` in the default Angular
`AppComponent`'s HTML:

`<h1>Hello Dockerized Angular App</h1>`

Now kill the running container, build a new image that will have version of the app, and run
a container from that image.

<pre>
dlwatts@terra somesite.com main* > docker kill somesite_v.1.0
somesite_v.1.0
dlwatts@terra somesite.com main* > docker build -t somesite.com:v.1.1 .
[+] Building 1.5s (13/13) FINISHED                                                         docker:default
 => [internal] load .dockerignore                                                                    0.1s
 => => transferring context: 2B                                                                      0.0s
 => [internal] load build definition from Dockerfile                                                 0.1s
 => => transferring dockerfile: 353B                                                                 0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                                      0.6s
 => [internal] load metadata for docker.io/library/node:latest                                       0.6s
 => [stage-1 1/2] FROM docker.io/library/nginx@sha256:67f9a4f10d147a6e04629340e6493c9703300ca23a2f7  0.0s
 => [builder 1/5] FROM docker.io/library/node@sha256:64b71834718b859ea389790ae56e5f2f8fa9456bf3821f  0.0s
 => [internal] load build context                                                                    0.5s
 => => transferring context: 3.33MB                                                                  0.4s
 => CACHED [builder 2/5] WORKDIR /usr/local/app                                                      0.0s
 => CACHED [builder 3/5] COPY ./ /usr/local/app                                                      0.0s
 => CACHED [builder 4/5] RUN npm install                                                             0.0s
 => CACHED [builder 5/5] RUN npm run build                                                           0.0s
 => CACHED [stage-1 2/2] COPY --from=builder /usr/local/app/dist/somesite.com /usr/share/nginx/html  0.0s
 => exporting to image                                                                               0.0s
 => => exporting layers                                                                              0.0s
 => => writing image sha256:4af578b452577243500071207f6f752b4faf1b8163dc4ab5e644a08911e02c3a         0.0s
 => => naming to docker.io/library/somesite.com:v.1.1                                                0.0s
dlwatts@terra somesite.com main* > docker run -d -p 8080:80 --name somesite_v.1.1 somesite.com:v.1.1
0b3283d8b23e97bf3ef0b0e7de1605676fedd3705853293896991e4a2ee11c24
dlwatts@terra somesite.com main* >
</pre>

Refresh the browser window to see the new version of the app. Now kill that container.
