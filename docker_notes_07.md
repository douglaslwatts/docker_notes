# Docker Notes 07

## Docker Registries

---

A registry is software that manage and distribute images. We can upload/download images to/from them.
The Docker company makes registries freely available.

### Finding images

---

We can use the `docker search` command to find images.

<pre>
dlwatts@terra ~ > docker search archlinux
NAME                                  DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
archlinux                             Arch Linux is a simple, lightweight Linux di…   494       [OK]
archlinux/archlinux                                                                   10
corpusops/archlinux                   archlinux corpusops baseimage                   0
corpusops/archlinux-bare              https://github.com/corpusops/docker-images/     0
ciready/archlinux                     CI-ready OpenSUSE images with CI-related bui…   0
archlinuxrolling/pacman                                                               0
archlinuxrolling/base                                                                 0
archlinuxjp/archlinux                                                                 1
archlinuxrolling/llvm                                                                 0
archlinuxrolling/bash                                                                 0
archlinuxrolling/gcc                                                                  0
archlinuxrolling/racket-minimal                                                       0
archlinuxrolling/git                                                                  0
archlinuxrolling/haskell                                                              0
archlinuxrolling/racket                                                               0
archlinuxrolling/scala                                                                0
archlinuxrolling/clojure                                                              0
base/archlinux                        Deprecated repository, use archlinux/base in…   322                  [OK]
blackikeeagle/archlinux-build         https://gitlab.com/herecura/archlinux-build     1
archlinuxjp/docker-arch                                                               0
archlinuxjp/archlinux-min                                                             0
immawanderer/archlinux                hourly updated archlinux base image             0
flamusdiu/archlinux-devel             Archlinux with devel and aura (https://githu…   1                    [OK]
aitorpazos/archlinux-docker-makepkg   A Docker image that allows building ArchLinu…   2                    [OK]
showgood163/archlinuxwithopenssh      latest archlinux with openssh support, EXPER…   1                    [OK]
dlwatts@terra ~ >
</pre>

More popular images have more stars. Images verified by Docker to be from the official organization
will have the OFFICIAL flag.

We also can search images online at [dockerhub](https://hub.docker.com). Here, we can get more
information about how to use the image, what its intended purpose is, and even see some examples
of docker files that use the image.

### Pushing Images

---

Once you create an account at dockerhub, you can push Images
to your own repositories. To get logged in via CLI, it is best to set up a credential helper. On
my Fedora box, I use `pass`.

`sudo dnf install pass`

`gpg2 --gen-key`

Now modify the permissions on the gnupg directory.

<pre>
dlwatts@terra ~ > ll .gnupg
total 16K
drwx------. 1 dlwatts dlwatts  176 Jul 27 22:09 openpgp-revocs.d/
drwx------. 1 dlwatts dlwatts  352 Jul 27 22:09 private-keys-v1.d/
-rw-r--r--. 1 dlwatts dlwatts 1.3K Jul 27 22:09 pubring.kbx
-rw-r--r--. 1 dlwatts dlwatts  658 Jul 27 21:23 pubring.kbx~
-rw-------. 1 dlwatts dlwatts  600 Jul 27 22:32 random_seed
-rw-------. 1 dlwatts dlwatts 1.4K Jul 27 22:29 trustdb.gpg
dlwatts@terra ~ > find .gnupg -type d -exec chmod 700 {} \;
dlwatts@terra ~ > find .gnupg -type f -exec chmod 600 {} \;
dlwatts@terra ~ > ll .gnupg
total 16K
drwx------. 1 dlwatts dlwatts  176 Jul 27 22:09 openpgp-revocs.d/
drwx------. 1 dlwatts dlwatts  352 Jul 27 22:09 private-keys-v1.d/
-rw-------. 1 dlwatts dlwatts 1.3K Jul 27 22:09 pubring.kbx
-rw-------. 1 dlwatts dlwatts  658 Jul 27 21:23 pubring.kbx~
-rw-------. 1 dlwatts dlwatts  600 Jul 27 22:32 random_seed
-rw-------. 1 dlwatts dlwatts 1.4K Jul 27 22:29 trustdb.gpg
dlwatts@terra ~ >
</pre>

Now we need to initialize pass with our public key from above. This will copy the public key value
into the password store's `.gpg-id` file. The `pub` line of the output from generating the key shows
the needed value or you can list the keys afterward.

<pre>
dlwatts@terra ~ > gpg2 -k
/home/dlwatts/.gnupg/pubring.kbx
--------------------------------
pub   ed25519 2023-07-28 [SC] [expires: 2025-07-27]
      5EXAMPLE19E336C120512F9F48428B391E08BD91638C
uid           [ultimate] dlwatts <example@example.com>
sub   cv25519 2023-07-28 [E] [expires: 2025-07-27]

dlwatts@terra ~ > pass init 5EXAMPLE19E336C120512F9F48428B391E08BD91638C
dlwatts@terra ~ > cat .password-store/.gpg-id
5EXAMPLE19E336C120512F9F48428B391E08BD91638C
dlwatts@terra ~ >
</pre>

Download the docker credential helper for your system architecture at
[docker-credential-helpers/releases](https://github.com/docker/docker-credential-helpers/releases).

For me that goes in my `~/Downloads/`, so I move it somewhere in `$PATH` and give it execute permissions.

<pre>
dlwatts@terra ~ > sudo mv Downloads/docker-credential-pass-v0.8.0.linux-amd64 /usr/local/bin
dlwatts@terra ~ > sudo chmod 755 /usr/local/bin/docker-credential-pass-v0.8.0.linux-amd64
dlwatts@terra ~ > ls -l /usr/local/bin/docker-credential-pass-v0.8.0.linux-amd64
-rwxr-xr-x. 1 dlwatts dlwatts 1.9M Jul 27 21:46 docker-credential-pass-v0.8.0.linux-amd64*
dlwatts@terra ~ >
</pre>

Now configure docker to use the credential helper. The value of `credsStore` should be the last
part of the filename of the credential helper file you downloaded. For me

<pre>
dlwatts@terra ~ > ls /usr/local/bin/docker-credential-pass-v0.8.0.linux-amd64
/usr/local/bin/docker-credential-pass-v0.8.0.linux-amd64
dlwatts@terra ~ > cat .docker/config.json
{
        "auths": {},
        "credsStore": "pass-v0.8.0.linux-amd64",
        "currentContext": "rootless"
}
dlwatts@terra ~ >
</pre>

Instructions for running Docker as a non-root user can be found at
[docs.docker.com/engine/security/rootless/](https://docs.docker.com/engine/security/rootless/).

Now lets login, tag an image, and push it to a repository.

<pre>
dlwatts@terra ~ > docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: dlwatts
Password:
Login Succeeded
dlwatts@terra ~ > cat .docker/config.json
{
        "auths": {
                "https://index.docker.io/v1/": {}
        },
        "credsStore": "pass-v0.8.0.linux-amd64",
        "currentContext": "rootless"
}
dlwatts@terra ~ > docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
archlinux     latest    1105a6ef0052   3 days ago      432MB
fedora        latest    ad2032316c26   3 weeks ago     190MB
nginx         latest    021283c8eb95   3 weeks ago     187MB
debian        latest    3676c78a12ad   3 weeks ago     116MB
ubuntu        latest    5a81c4b8502e   4 weeks ago     77.8MB
hello-world   latest    feb5d9fea6a5   22 months ago   13.3kB
ubuntu        14.04     13b66b487594   2 years ago     197MB
dlwatts@terra ~ > docker tag archlinux dlwatts/archlinux-server:v.1.0
dlwatts@terra ~ > docker push dlwatts/archlinux-server:v.1.0
The push refers to repository [docker.io/dlwatts/archlinux-server]
667877c95a48: Pushed
e3946f2a3c3f: Mounted from library/archlinux
v.1.0: digest: sha256:e21266b93a65192313d3857651abf2b5a194bad9438b3d8a565aeeb3b4400d5e size: 738
dlwatts@terra ~ > docker images
REPOSITORY                 TAG       IMAGE ID       CREATED         SIZE
dlwatts/archlinux-server   v.1.0     1105a6ef0052   3 days ago      432MB
archlinux                  latest    1105a6ef0052   3 days ago      432MB
fedora                     latest    ad2032316c26   3 weeks ago     190MB
nginx                      latest    021283c8eb95   3 weeks ago     187MB
debian                     latest    3676c78a12ad   3 weeks ago     116MB
ubuntu                     latest    5a81c4b8502e   4 weeks ago     77.8MB
hello-world                latest    feb5d9fea6a5   22 months ago   13.3kB
ubuntu                     14.04     13b66b487594   2 years ago     197MB
dlwatts@terra ~ >
</pre>

Now I have a repo called archlinux-server in my namespace with the image I just pushed. You can see
your repos by logging in at dockerhub.
