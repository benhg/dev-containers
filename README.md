# SAPEON SW developer environment

## Introduction

This repo is for the development environment for on-card software. It contains:

1. This README
2. Containerfiles specifying required packages for building/testing code on Ubuntu 22 and Rocky Linux

## Prerequisites

- Add a GitHub SSH key or personal access token
- Authorize it on the SAPEON-AI organization
- Be added to the SAPEON-AI organization on Github
- Clone this repo

## Building a container

There are two containerfiles included in this repository - they are named `Containerfile_[os]`. To build them on your local machine, you can run the command `docker build -f <file_name> -t <some tag>`.

The <some tag> should be descriptive enough to remind you what container is for what. You will use it to run your containers.

After building the container, you should see some output like this:

```
benglick@Bens-MacBook-Pro sw_env % docker build -f Containerfile_ubuntu . -t ubuntu_dev
[+] Building 0.4s (8/8) FINISHED                                                                                                                                                  docker:desktop-linux
 => [internal] load build definition from Containerfile_ubuntu                                                                                                                                    0.0s
 => => transferring dockerfile: 338B                                                                                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                                                                                 0.0s
 => => transferring context: 2B                                                                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/ubuntu:22.04                                                                                                                                   0.3s
 => [1/4] FROM docker.io/library/ubuntu:22.04@sha256:8eab65df33a6de2844c9aefd19efe8ddb87b7df5e9185a4ab73af936225685bb                                                                             0.0s
 => CACHED [2/4] RUN apt update                                                                                                                                                                   0.0s
 => CACHED [3/4] RUN apt install -y linux-headers-5.15.0-89-generic                                                                                                                               0.0s
 => CACHED [4/4] RUN echo 'export PS1="[\u@ubuntu_container] \W # "' >> /root/.bash_profile                                                                                                       0.0s
 => exporting to image                                                                                                                                                                            0.0s
 => => exporting layers                                                                                                                                                                           0.0s
 => => writing image sha256:e882642bb222fffd6d98e035428437a72225442f5a6a9dff7da7fb280a16aeb5                                                                                                      0.0s
 => => naming to docker.io/library/ubuntu_dev                                                                                                                                                     0.0s

What's Next?
  1. Sign in to your Docker account → docker login
  2. View a summary of image vulnerabilities and recommendations → docker scout quickview
```

You can then run `docker images` to list local images:

```
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
ubuntu_dev   latest    e882642bb222   29 seconds ago   201MB
```


## Running a container

After your image is built, you can start using it. To run the container, you will use the `docker run` command.

The basic run command syntax is `docker run [options] <container tag>`. The most useful options are `-it`, which tells Docker to run your container in interactive mode and `-v`, which is used for bind-mounting files and directories into the container.

Directories that are bind-mounted into the container are accessible both from the container and from the host OS. To bind-mount a directory, add `-v<host_path>:<container_path>` to your `docker run` command.

For example, the command:

```
benglick@Bens-MacBook-Pro sw_env % docker run -v/Users/benglick:/mnt/home -it ubuntu_dev 
[root@ubuntu_container] / # 
```

Results in creating an Ubuntu-like shell where you can use all the Linux commands you'd normally expect.

Additionally, since the `-v/Users/benglick:/mnt/home` flag was passed, files in my home directory on my laptop at `/mnt/home` are accessible inside the container:

Note that the `-v` arguments (and any other named parameters) need to come before the tag.

```
[root@ubuntu_container] / # ls /mnt/home/Desktop/
sw_env  test-design-docs  test_p2pdma_module  x300_kernel_driver
```

If I run `touch /mnt/home/Desktop/written_by_container` from the container, and then `ls /Users/benglick/Desktop` from the host, I see:

```
benglick@Bens-MacBook-Pro log % ls /Users/benglick/Desktop | grep written_by_container
written_by_container
```

To start additional processes in the existing container, first find the running container's ID with `docker ps`:

```
benglick@Bens-MacBook-Pro log % docker ps
CONTAINER ID   IMAGE        COMMAND       CREATED         STATUS         PORTS     NAMES
830222986bbc   ubuntu_dev   "/bin/bash"   2 seconds ago   Up 2 seconds             modest_rosalind
```

With that, you can use `docker exec` to start additional processes:

```
benglick@Bens-MacBook-Pro log % docker exec -it 830222986bbc /bin/bash
[root@ubuntu_container] / # 
```
