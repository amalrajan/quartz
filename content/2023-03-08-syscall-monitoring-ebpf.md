---
title: Syscall tracing with eBPF
date: 2023-03-08
---

In this post, I'll demonstrate how you can distinguish kernel system calls made by processes within a container using eBPF (extended Berkeley Packet Filter) programs.


## PID Isolation in Containers

PID namespaces are integral to Linux kernel namespaces, designed to provide each group of processes with a unique set of process IDs. This allows processes within a container to function in a separate, isolated environment. The segregation achieved by PID namespaces improves process management and resource allocation by limiting interactions between processes. This feature is critical in container technologies like Docker and Kubernetes for maintaining process isolation.


![PID Isolation in Containers](https://ik.imagekit.io/5jrct2yttdr/quartz/Drawing%202026-02-18%2020.59.07.excalidraw_Nvg_fA2-H.png)

  

## Determining PID Mapping in Docker Containers

To identify process ID (PID) mapping within Docker containers, follow these steps:

  

Retrieve the full image ID of the active container. ![Determining PID Mapping in Docker Containers](https://ik.imagekit.io/5jrct2yttdr/amalrajan.github.io/Screenshot%202023-03-09%20202810_BYIDI4s2_.png?updatedAt=1714870132034)

  

Use the docker top command to view the PID and parent PID (PPID) mapping. ![Determining PID Mapping in Docker Containers](https://ik.imagekit.io/5jrct2yttdr/amalrajan.github.io/Screenshot%202023-03-09%20201313_1nwZ-fu9D.png?updatedAt=1714870132051)

  

If the above step is insufficient, navigate to the directory `/sys/fs/cgroup/unified/docker/<long-image-id>/cgroup.procs` to view PPIDs. You ca write a simple shell script to map these PIDs.

  

## Using BCC Tools with PID Parameters

The BPF Compiler Collection (BCC) is a toolkit for developing and executing eBPF (extended Berkeley Packet Filter) programs. BCC provides a variety of pre-built eBPF programs and libraries suitable for system performance monitoring, syscall tracing, and more. BCC tools commonly support the -p flag, allowing users to specify a PID for targeted monitoring. These tools are instrumental in gaining insights into system operations and enhancing security measures within containerized environments.

  

## Bonus

If you're facing trouble setting up BCC tools on your system, consider using my [Docker image](https://hub.docker.com/r/amalrajan/ubuntu-bcc) with pre-installed BCC tools. This image is designed to simplify the process of monitoring syscalls within Docker containers.

  

Make sure to launch it with
```bash
docker run -it -d --privileged -v /lib/modules:/lib/modules -v /sys:/sys -v /usr/src:/usr/src amalrajan/ubuntu-bcc:focal
```

This command initiates a detached container with privileged access, necessary for syscall monitoring, and mounts the required directories to ensure the container has access to the host's kernel modules and source directories.