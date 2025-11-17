---
title: A Practical Introduction to Containers with Docker
date: 2025-11-17 00:00:00 -0500
---

A container is an isolated Linux process running on a host Linux operating system. A container has its own hostname, root filesystem, process IDs, mountpoints, user IDs etc. independent of the host OS, so it acts like a new operating system with its own environment. However, all containers in a host use the host's Linux kernel. This means [system calls](https://man7.org/linux/man-pages/man2/syscalls.2.html), which are a kernel's job to execute, like [running a program](https://man7.org/linux/man-pages/man2/execve.2.html) or [creating a new directory](https://man7.org/linux/man-pages/man2/mkdir.2.html), are done by the host Linux kernel for all containers.

If this sounds awfully similar to a virtual machine, it is. The only difference is that a virtual machine simulates the hardware of a physical machine, whereas a container only simulates the user space—the non-kernel portion of an operating system. A container does not simulate the kernel. So if your host uses the Linux kernel, it cannot run Windows or FreeBSD containers since they require separate kernels. This is not a limitation for virtual machines, where each virtual machine can have a separate kernel installed.

This dependence on the host kernel makes containers blazing fast to deploy and remove, especially compared to virtual machines.

One way to run containers is with program called Docker Engine. Docker is used by developers to deploy their own web apps. Homelab hobbyists use Docker to deploy popular web apps like [Pi-Hole](https://pi-hole.net/) and [Jellyfin](https://jellyfin.org/).

For a truly practical understanding of containers, you will need to install Docker to follow the rest of the article:
- [Install Docker Engine on Linux](https://docs.docker.com/engine/install) and follow [post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/)
- [Install Docker Desktop on Mac](https://docs.docker.com/desktop/setup/install/mac-install/) (uses Linux VM)
- [Install Docker Desktop on Windows](https://docs.docker.com/desktop/setup/install/windows-install/) (uses WSL)

Now open a terminal and spin up a container that mimick's Ubuntu's user space.
```
docker run ubuntu cat /etc/os-release
```
This command deploys an Ubuntu container and runs the command `cat /etc/os-release` inside it. Inside the container, `os-release` shows the OS as Ubuntu regardless of what the host OS is. Immediately when this command stops running, the container stops.

You can check this yourself:
```
docker ps -a
```
This will print something like this:
```
CONTAINER ID   IMAGE     COMMAND                 CREATED          STATUS                        PORTS     NAMES
6b8dc0691a1e   ubuntu    "cat /etc/os-release"   2 minutes ago    Exited (0) 2 minutes ago                agitated_ganguly
```
It shows the ID of the container you ran, what command you specified, and that it exited 2 minuted ago. There is also a randomly generated name for the container, `agitated_ganguly`.

How can you prevent a container from closing? Let's run a command like `bash` that keeps running until you close it.
```
docker run -ti ubuntu /bin/bash
```
The `-ti` here is necessary since we want bash to run as an interactive terminal.

Voila, now you're inside the container. Notice how the hostname is different from your host OS. You can use `ls` and `cd` to roam around the container and run `exit` to get out of it. Notice that the container is stopped the moment you exit `bash`, i.e. when `bash` stops running.

But you don't always need to specify a command when running Docker containers. Here's an example of a web app container: https://github.com/docker/welcome-to-docker. The repo contains the code for the web app that a developer would write. However, there is also code for hosting the web app inside a Docker container in [Dockerfile](https://github.com/docker/welcome-to-docker/blob/main/Dockerfile). One thing interesting to note is the last line, where a command is specified with `CMD` to run the HTTP server for the web app.

Let us deploy this container:
```
docker run -p 8088:80 docker/welcome-to-docker
```
`-p 8080:80` commands Docker to map the container's port 80 to the host's port 8080. So now you can visit http://localhost:8080 to see the web app the container is hosting.

Notice that you did not have to specify a command this time because it was already specified in the Dockerfile.
