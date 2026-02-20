---
title: Syscall tracing with eBPF
date: 2023-03-08
---
In this post, I'll demonstrate how you can distinguish between kernel system calls made by processes in a multi-container environment using eBPF (extended Berkeley Packet Filter).

## PID Isolation in Containers

PID isolation is one of the foundations on top of which Docker containers are built. In the diagram below, processes P3.1, P3.2 and P3.3 are in a “child namespace”. These processes are isolated from the other processes, and cannot see them.
On the other hand, the reverse is true — processes from the parent namespace can absolutely see those in the child namespace. There is a catch however, the child namespace PIDs are named differently in the context of parent namespace. For example, P3.1 could be seen as P2940 by a process in the parent namespace.

![PID Isolation in Containers](https://ik.imagekit.io/5jrct2yttdr/quartz/Drawing%202026-02-18%2020.59.07.excalidraw_Nvg_fA2-H.png)


## Setting up the test environment

The best way I would suggest is to write a `docker-compose.yml` to spin up a few containers.
You could then manually SSH into each of them and trigger a file system write operation or automate that as well.
## Determining PID Mapping in Docker Containers

To identify process ID (PID) mapping within Docker containers, follow these steps:
Retrieve the full image ID of the active container. ![Determining PID Mapping in Docker Containers](https://ik.imagekit.io/5jrct2yttdr/amalrajan.github.io/Screenshot%202023-03-09%20202810_BYIDI4s2_.png?updatedAt=1714870132034)

Use the docker top command to view the PID and parent PID (PPID) mapping. ![Determining PID Mapping in Docker Containers](https://ik.imagekit.io/5jrct2yttdr/amalrajan.github.io/Screenshot%202023-03-09%20201313_1nwZ-fu9D.png?updatedAt=1714870132051)

  If the above step is insufficient, navigate to the directory `/sys/fs/cgroup/unified/docker/<long-image-id>/cgroup.procs` to view parent PIDs. You can write a simple shell script to map these PIDs.

## Using BCC Tools with PID Parameters

The BPF Compiler Collection (BCC) is a toolkit for developing and executing eBPF (extended Berkeley Packet Filter) programs. BCC provides a variety of pre-built eBPF programs and libraries suitable for system performance monitoring, syscall tracing, and more. BCC tools commonly support the -p flag, allowing users to specify a PID for targeted monitoring. These tools are instrumental in gaining insights into system operations and enhancing security measures within containerized environments.

## Bonus

If you're facing trouble setting up BCC tools on your system, consider using my [Docker image](https://hub.docker.com/r/amalrajan/ubuntu-bcc) with pre-installed BCC tools. This image is designed to simplify the process of monitoring syscalls within Docker containers.

Make sure to launch it with
```bash
docker run -it -d \
  --privileged \
  -v /lib/modules:/lib/modules \
  -v /sys:/sys \
  -v /usr/src:/usr/src \
  amalrajan/ubuntu-bcc:focal
```

This command initiates a detached container with privileged access, necessary for syscall monitoring, and mounts the required directories to ensure the container has access to the host's kernel modules and source directories.