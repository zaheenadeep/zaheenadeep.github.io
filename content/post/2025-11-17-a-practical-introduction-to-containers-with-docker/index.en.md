---
title: A Practical Introduction to Containers with Docker
date: 2025-11-17 00:00:00 -0500
---

### What is a Container?

A container is an _isolated_ Linux process running on a Linux-based operating system. The keyword here is _isolated_. A container has its own hostname, root filesystem, process IDs, mountpoints, and user IDs independent of the host OS. Even though a container is only a Linux process, this isolation of attributes from the host OS makes a container appear as a separate operating system with its own files, users, and network interfaces.

### Containers vs Virtual Machines

If a container sounds awfully similar to a virtual machine, that is because it is. The difference is that a virtual machine simulates the hardware of a physical machine (including its motherboard, CPU, RAM, and NIC), whereas a container only simulates the _user space_ of an operating system.

What is this user space? An operating system consists of two parts: kernel and user space. The kernel runs with higher privileges and conducts core operating system tasks like memory management, process creation, and block I/O management. The user space consists of every other program in the operating system that is not the kernel. This includes applications like `bash`, `ip`, or even Chrome. A container only simulates user space and not the kernel because, as a Linux process, it can simply use the Linux kernel of the host OS.

So a host OS can have multiple containers, but they will all share the host OS kernel. This makes containers blazing fast to deploy, because you're skipping the overhead of simulating the kernel (or hardware—as in the case with virtual machines).

### Docker

One way to run containers is with a program called Docker. Docker is used by developers for creating local dev environments to deploy their applications inside containers. Homelab hobbyists use Docker to deploy popular web apps like [Pi-Hole](https://pi-hole.net/) and [Jellyfin](https://jellyfin.org/).

For a practical understanding of containers, you will need to install Docker to follow the rest of the article:
- [Install Docker Engine on Linux](https://docs.docker.com/engine/install) and follow [post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/)
- [Install Docker Desktop on Mac](https://docs.docker.com/desktop/setup/install/mac-install/) (uses Linux VM)
- [Install Docker Desktop on Windows](https://docs.docker.com/desktop/setup/install/windows-install/) (uses WSL)

### Demo 1 – Print OS Info

Now open a terminal and spin up a container that mimick's Ubuntu's user space.
```shell
docker run ubuntu cat /etc/os-release
```
This command deploys an Ubuntu container and runs the command `cat /etc/os-release` inside it. Inside the container, `os-release` shows the OS as Ubuntu regardless of what the host OS is. Immediately when this command stops running, the container stops.

You can check this yourself:
```shell
docker ps -a
```
This will print something like the following:
```
CONTAINER ID   IMAGE     COMMAND                 CREATED          STATUS                        PORTS     NAMES
6b8dc0691a1e   ubuntu    "cat /etc/os-release"   2 minutes ago    Exited (0) 2 minutes ago                agitated_ganguly
```
It shows the ID of the container you ran, what command you specified, and when the container exited. There is also a randomly generated name for the container you can use for identification in future Docker commands. In this case, the name is `agitated_ganguly`.

### Demo 2 – Interactive Shell

But this container exited the moment the command stopped running. To prevent the container from exiting, let's run a command like `bash` that keeps running until you explicitly close it.
```shell
docker run -ti ubuntu /bin/bash
```
> The `-t` and `-i` options are necessary here since we want bash to run as an interactive terminal.
{ .prompt-info }

Voila, now we're inside the container. Notice how the hostname of the container is different from the host OS according to the bash prompt. You can use `ls` and `cd` to roam around inside the container and run `exit` to get out of it. The container will stop the moment you exit `bash`, i.e. when the command specified with `docker run` stops.

### Demo 3 – Test Web App Deployment

But you don't always need to specify a command when running Docker containers.

Here's an example:
https://github.com/docker/welcome-to-docker

The repo contains code for a web app. However, there is also code for building a Docker container in [Dockerfile](https://github.com/docker/welcome-to-docker/blob/main/Dockerfile).

Important things to note in this file:
1. The container is built on top of an existing container image called `node-21:alpine`. This is an [Alpine Linux container with NodeJS installed](https://hub.docker.com/_/node).
2. In the last line, a `docker run` command is specified with `CMD`. The command runs the HTTP server. This means we won't need to specify a command when we run `docker run` for this container.

Let us deploy this container without a command to see it in action:
```shell
docker run -p 8080:80 docker/welcome-to-docker
```
`-p 8080:80` tells Docker to map the container's port 80 to the host's port 8080. This allows you to access the container's port 80, which the HTTP server inside the container is listening on, by visiting http://localhost:8080 from the host. Try it with a web browser or `curl`.

Once done, you can stop the container by sending SIGINT (Ctrl + C).

### Demo 4 – Deploying Joplin with Docker Compose

Finally, let's deploy an actually useful app. But instead of deploying with `docker run`, use `docker compose`. This approach is more popular in production and among homelabbers.

We will deploy Joplin, an open-source note-taking application. First, take a look at the `docker run` command [here](https://docs.linuxserver.io/images/docker-joplin/#docker-cli-click-here-for-more-info). The multiline Docker CLI command is not pleasant to read or understand.

So instead we will make a Compose file with the equivalent YAML configuration [here](https://docs.linuxserver.io/images/docker-joplin/#docker-compose-recommended-click-here-for-more-info). First, make a directory named `joplin`. Then create a file inside named `compose.yaml` with the [Compose configuration](https://docs.linuxserver.io/images/docker-joplin/#docker-compose-recommended-click-here-for-more-info). Replace `/path/to/config` with `./config` for the sake of the demo. You would normally point this to the path in your host OS where you want Joplin's persistent configurations to be saved.

Now with `joplin` as your current working directory, run
```shell
docker compose up -d
```
> Option `-d` runs the container in background, i.e. in detached mode. This option exists for `docker run` as well.
{ .prompt-info }

This will deploy the container (to be precise, the "service") defined in the Compose file.

Visit https://localhost:3001 and voila, you have your own note-taking application accessible with a browser!

If ever needed, you can gracefully undeploy the container by running
```shell
docker compose down
```
from the `joplin` directory where `compose.yaml` resides.

This will not delete the `./config` directory where Joplin stores the notes you create. So if you run `docker compose up -d` again to deploy a new Joplin container, your notes will still be there. That is the magic of containers!

### Wrapping Up

You've learned how to use Docker CLI and its better variant Docker Compose to spin up containers. Now it's your turn to deploy an app _you_ want to use using Docker! Check out [this awesome list](https://github.com/awesome-selfhosted/awesome-selfhosted) and deploy an app you like. Whatever app you choose, chances are it officially supports Docker-based installation.

### More Resources

- [Komodo](https://komo.do/docs/intro): a web UI for Docker to make life easier
- [Kubernetes](https://youtu.be/BE77h7dmoQU): learn about the most popular container orchestrator used in production deployments