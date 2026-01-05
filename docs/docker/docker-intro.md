# Docker Introduction

- In Linux, the kernel is the central component of the operating system, acting as a bridge between hardware and software.
- It manages system resources such as the CPU, memory, and devices, and handles core functions like running programs, managing files, and controlling input/output operations.

--- 

## Docker vs VirtualMachine
 - This is understandable, but incorrect, comparison. The biggest difference is that **virtual machines** virtualize hardware whereas **containers** virtualize operating system kernels.
 - Virtual machines run on a platform called a hypervisor. The hypervisor's main job is to translate operations on emulated hardware within virtual machines like memory processors, disks, etc, to operations on real hardware within their hosts.
    - Run on top of hypervisors
    - Need hardware emulation
    - Require OS Config
    - Can run many apps at once
    - Take up more spaces
 - Containers run on container run times. Container run times work with the operating system to allocate hardware and copy files and directories, including the parts with your application in it into something that looks more like any other app running on that system.
    - Run in container runtimes
    - Work alongside OS
    - Do not require OS Config
    - Run one app at a time (usually)
    - Take up less spaces

---

## Docker Anatomy 
- A container is composed of two things: a Linux namespace (Host) and a Linux control group (Kernel).
- Namespaces are a Linux kernel feature that provides the ability to expose different "views" of your system to applications running within it.
- Namespace limits "What you can see".
- Control Groups provide the ability to restrict how much hardware (CPU, Network and Disk bandwidth, Memory) each process can use. "Assign disk quotas" not supported by Docker.
- Docker uses control groups for a few things.
    - First, it uses control groups to monitor and restrict CPU usage, or the amount of CPU time each container can take up.
    - Second to monitor and restrict network and disk bandwidth. And more commonly, Docker uses control groups to monitor and restrict memory consumption. This makes it easy to prevent busier, or larger containers from eating up all the system's resources
- Control group limits "How much you can see".
- Today's Linux kernels provide eight namespaces. <br />
    1. USERNS - User management <br />
    2. MOUNT - Access File system <br />
    3. NET - Network <br />
    4. IPC - Inter Process Communication - resources <br />
    5. TIME (Not support by Docker due to Technical limitation) <br />
    6. PID - Process ID <br />
    7. CGROUP - Create control group <br />
    8. UTS - Unix Time Stamp <br />
- Control groups, another Linux kernel feature, build on this by providing the ability to restrict how much hardware each process can use.
- Control groups and Namespaces are Linux only features, this means that Docker technically only runs natively on Linux and some newer versions of Windows as well.
- Containers can run on anything but their images are tied to the kernel they were created from.
- Containers created from container images configured for Linux kernels can only run on Linux. Conversely, containers created from container images configured for Windows can only run on Windows. 

---

## Docker Advantages

1- Docker Files make configuration and packaging apps and their environments really easy. <br />
2- Docker Hub (Global repo of images maintained by Docker) makes sharing images with anyone in the world easy. <br />
3- Docker CLI makes it really easy to start the apps in containers. <br />


### Docker Alternatives 
- Kuberbetes
- Podman : developed by Red Hat to provide a more secure, daemon-less approach to container management. Supports rootless containers and can run multiple apps at once with an "init" system.
  - It was created to address some concerns around Docker:
    - Security: Eliminates the need for a privileged daemon (reduces attack surface).
    - Integration with systemd: Containers can run as regular system services.
    - Kubernetes alignment: Uses the same OCI runtime/spec as Kubernetes (CRI-O).

---
