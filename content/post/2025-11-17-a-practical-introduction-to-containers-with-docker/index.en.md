---
title: A Practical Introduction to Containers with Docker
date: 2025-11-17 00:00:00 -0500
---

### What is a Container?

A container is an _isolated_ Linux process running on a Linux-based host operating system. The keyword here is _isolated_. A container has its own hostname, root filesystem, process IDs, mountpoints, and user IDs independent of the host OS. Even though a container is only a Linux process, this isolation of attributes from the host OS makes a container appear as a separate operating system with its own files, users, and network interfaces.

### Containers vs Virtual Machines

If a container sounds awfully similar to a virtual machine, that is because it is. The difference is that a virtual machine simulates the hardware of a physical machine (including its motherboard, CPU, RAM, and NIC), whereas a container only simulates the _user space_ of an operating system.

What is this user space? An operating system consists of two parts: user space and kernel. The kernel runs with higher privileges and conducts core operating system tasks like memory management, process creation, block I/O management, and network implementation. The user space is everything else in the operating system, including applications like `bash` and `ip`. A container only simulates user space because, as a Linux process, it uses the host operating system's Linux kernel.

So a host OS can have multiple containers, but they will all use host OS kernel. This makes containers blazing fast to deploy and remove, because you're skipping the overhead of simulating the kernel or hardware—as in the case of a virtual machine.

### Docker

One way to run containers is with a program called Docker. Docker is used by developers to deploy the web apps they develop. Homelab hobbyists use Docker to deploy popular web apps like [Pi-Hole](https://pi-hole.net/) and [Jellyfin](https://jellyfin.org/).

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
It shows the ID of the container you ran, what command you specified, and that it exited 2 minuted ago. There is also a randomly generated name for the container, `agitated_ganguly`.

### Demo 2 – Interactive Shell

But this container closed the moment we ran the command. To prevent the container from closing, let's run a command like `bash` that keeps running until you close it.
```shell
docker run -ti ubuntu /bin/bash
```
> The `-ti` here is necessary since we want bash to run as an interactive terminal.
{ .prompt-info }

Voila, now we're inside the container. Notice how the hostname is different from the host OS. You can use `ls` and `cd` to roam around the container and run `exit` to get out of it. The container will stop the moment you exit `bash`, i.e. when the specified command stops running.

### Demo 3 – Web App Deployment

But you don't always need to specify a command when running Docker containers. Here's an example of a web app container: https://github.com/docker/welcome-to-docker. The repo contains the code for a web app that a developer might write. However, there is also code for hosting the web app inside a Docker container in [Dockerfile](https://github.com/docker/welcome-to-docker/blob/main/Dockerfile). One thing interesting to note is the last line, where a command is specified with `CMD` to run the HTTP server for the web app.

Let us deploy this container:
```shell
docker run -p 8080:80 docker/welcome-to-docker
```
`-p 8080:80` tells Docker to map the container's port 80 to the host's port 8080. This allows you to access the container's port 80, where the HTTP server is listening, by visiting http://localhost:8080 from the host's web browser.

> You did not have to specify a command this time because `CMD` was already specified in the Dockerfile.
{ .prompt-info }

You can stop the container by sending SIGINT (Ctrl + C). 

### Demo 4 – Docker Compose

Options to `docker run` can get very long. As an example, check out 
