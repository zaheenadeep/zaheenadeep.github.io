---
title: A Practical Introduction to Containers with Docker
date: 2025-11-17 00:00:00 -0500
---

A container is an isolated Linux process running on a host Linux operating system. A container has its own hostname, root filesystem, process IDs, mountpoints, user IDs etc. independent of the host OS, so it acts like a new operating system with its own environment. However, all containers in a host use the host's Linux kernel. This means [system calls](https://man7.org/linux/man-pages/man2/syscalls.2.html), which are a kernel's job to execute, like [running a program](https://man7.org/linux/man-pages/man2/execve.2.html) or [creating a new directory](https://man7.org/linux/man-pages/man2/mkdir.2.html), are done by the host Linux kernel for all containers.

If this sounds awfully similar to a virtual machine, it is. The only difference is that a virtual machine simulates the hardware of a physical machine, whereas a container only simulates the user space—the non-kernel portion of an operating system. A container does not simulate the kernel. So if your host uses the Linux kernel, it cannot run Windows or FreeBSD containers since they require separate kernels. This is not a limitation for virtual machines, where each virtual machine can have a separate kernel installed.

This dependence on the host kernel makes containers blazing fast to deploy and remove, especially compared to virtual machines.

One way to run containers is with program called Docker Engine. Docker is used by developers to deploy their own web apps. Homelab hobbyists use Docker to deploy popular web apps like [Pi-Hole](https://pi-hole.net/) and [Jellyfin](https://jellyfin.org/).

For a truly practical understanding of containers, you will need to install Docker to follow the rest of the article:
- [Install Docker Engine on Linux](https://docs.docker.com/engine/install)
- [Install Docker Desktop on Mac](https://docs.docker.com/desktop/setup/install/mac-install/) (uses Linux VM)
- [Install Docker Desktop on Windows](https://docs.docker.com/desktop/setup/install/windows-install/) (uses WSL)
