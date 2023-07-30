# Docker Notes 08

## Dockerfile ([Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)) ([Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/))

---

- A Dockerfile is a small program that describes how to build a Docker image.
- To build the image, use the `docker build` command.

  Example:

  `docker build -t name-of-resulting-image ./relative/path/to/DockerfileLocation/`

- When the program finishes executing successfully, the resulting image will be available
  in your local docker registry.
- Each line starts with the image built in the previous line, runs a container in which changes are
  made, and makes another image from that container.
- The previous image is unchanged.
- The state is not carried forward from line to line.
- Only download packages and files that are truly needed into your image to keep is as small as
  possible, and to keep build times down.
- The results from each step/line in a Dockerfile is cached.
  - Subsequent builds only run a step if there are changes that should effect the
    result of that step.
  - If a file is downloaded into the image, it will not be downloaded in subsequent runs of the
    Dockerfile so we must explicitly tell Docker if it needs a new version to be downloaded.
- Put the parts of a Dockerfile that need to change most often at the end of the Dockerfile so that
  we run as few steps on subsequent builds as possible.
- A Dockerfile is not like a shell scripts, processes started on one line are not running on the next
  line. If one program needs to start and then another in sequence, then those two operations must
  occur on the same line.
- If we use the ENV command, then environment variables can be set and persist throughout the
  Dockerfile. This causes the variable to be set on each step.

  NOTE: Each step/line is a call to `docker run ...` and then to `docker commit ...`

| Statement &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Use                                                                                                      |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------- |
| MAINTAINER first last \<user@email.com\>                                                                                                                                                                                                                                                                                                                                | Define the author of the Dockerfile                                                                      |
| FROM image_name                                                                                                                                                                                                                                                                                                                                                         | Specify and image to start from                                                                          |
| RUN command_to_run                                                                                                                                                                                                                                                                                                                                                      | Specify a command to run                                                                                 |
| ADD file_archive_or_url path                                                                                                                                                                                                                                                                                                                                            | Adds the file, archive contents, or the contents at the url to the path in the image                     |
| ENV VAR_NAME=value                                                                                                                                                                                                                                                                                                                                                      | Set an environment variable that will persist throughout the Dockerfile and exist in the resulting image |
| ENTRYPOINT command                                                                                                                                                                                                                                                                                                                                                      | Set a command that args can be passed to when running a container from the resulting image               |
| CMD \["/path/to/command", "command_arg"\]                                                                                                                                                                                                                                                                                                                               | Command to run inside the container                                                                      |
| CMD /path/to/command arg_to_command                                                                                                                                                                                                                                                                                                                                     | Command to run inside the container                                                                      |
| EXPOSE port_number                                                                                                                                                                                                                                                                                                                                                      | Expose the port into the container                                                                       |
| VOLUME \["/host/path/", "/container/path/"\]                                                                                                                                                                                                                                                                                                                            | Share a directory between the host and the container                                                     |
| VOLUME \["/shared-data"\]                                                                                                                                                                                                                                                                                                                                               | Make a directory which can be shared with other containers                                               |
| WORKDIR directory                                                                                                                                                                                                                                                                                                                                                       | Set the working directory for the remainder of the Dockerfile and inside the container                   |
| USER username                                                                                                                                                                                                                                                                                                                                                           | Run the rest of the commands as the username specified                                                   |

For a simple example, create the following python script and Dockerfile:

#### print_todos.py

---

<pre>
#!/usr/sbin/python

todo_lst = []
while 'q' not in todo_lst:
    todo_lst.append(input('Enter a ToDo > ').strip())

todo_dict = {count + 1: todo for (count, todo) in enumerate(todo_lst) if todo != 'q'}

for todo_num, todo in todo_dict.items():
    print('{:3}: {}'.format(todo_num, todo))
</pre>

#### Dockerfile

---

<pre>
# Start with official archlinux image
FROM archlinux

# Update and install python in the intermediate image
RUN pacman -Syu python --noconfirm

# Add the python script into the next intermediate image
ADD ./print_todos.py /print_todos.py

# Make the script executable in the next intermediate image
CMD chmod 700 /print_todos.py

# Add the command to run the script in the final image
CMD /print_todos.py

</pre>

Now build the image and run a container using it.

<pre>
dlwatts@terra ~ > docker build -t todo_py .
[+] Building 0.4s (8/8) FINISHED                                                           docker:default
 => [internal] load build definition from Dockerfile                                                 0.1s
 => => transferring dockerfile: 515B                                                                 0.0s
 => [internal] load .dockerignore                                                                    0.1s
 => => transferring context: 2B                                                                      0.0s
 => [internal] load metadata for docker.io/library/archlinux:latest                                  0.0s
 => [1/3] FROM docker.io/library/archlinux                                                           0.0s
 => [internal] load build context                                                                    0.1s
 => => transferring context: 95B                                                                     0.0s
 => CACHED [2/3] RUN pacman -Syu python --noconfirm                                                  0.0s
 => CACHED [3/3] ADD ./print_todos.py /print_todos.py                                                0.0s
 => exporting to image                                                                               0.0s
 => => exporting layers                                                                              0.0s
 => => writing image sha256:fe88d92ca64ee52d61875625bb05fd9c34d4eba099c39d34ba20da774d793792         0.0s
 => => naming to docker.io/library/todo_py                                                           0.0s
dlwatts@terra ~ > docker run --rm -ti todo_py
Enter a ToDo > Have breakfast
Enter a ToDo > Go for a hike
Enter a ToDo > Play some music
Enter a ToDo > q
  1: Have breakfast
  2: Go for a hike
  3: Play some music
dlwatts@terra ~ >
</pre>
