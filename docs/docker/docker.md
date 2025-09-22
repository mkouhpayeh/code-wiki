# Docker

## Docker vs VirtualMachine
 - This is understandable, but incorrect, comparison. The biggest difference is that **virtual machines** virtualize hardware whereas **containers** virtualize operating system kernels.
 - Virtual machines run on a platform called a hypervisor. The hypervisor's main job is to translate operations on emulated hardware within virtual machines like memory processors, disks, etc, to operations on real hardware within their hosts.
    - Run on top of hypervisors
    - Need hardware emulation
    - Require OS Config
    - Can run many apps at once
 - Containers run on container run times. Container run times work with the operating system to allocate hardware and copy files and directories, including the parts with your application in it into something that looks more like any other app running on that system.
    - Run in container runtimes
    - Work alongside OS
    - Do not require OS Config
    - Run one app at a time (usually)
 
## Docker Anatomy 
- A container is composed of two things: a Linux namespace and a Linux control group.
- Namespaces are a Linux kernel feature that provides the ability to expose different "views" of your system to applications running within it.
- Today's Linux kernels provide eight namespaces.
- Control groups, another Linux kernel feature, build on this by providing the ability to restrict how much hardware each process can use. Docker uses control groups for a few things.
- Control groups and Namespaces are Linux only features, this means that Docker technically only runs natively on Linux and some newer versions of Windows as well.
- Containers can run on anything but their images are tied to the kernel they were created from.
- Containers created from container images configured for Linux kernels can only run on Linux. Conversely, containers created from container images configured for Windows can only run on Windows. 
