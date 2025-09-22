# Docker

## Docker vs VirtualMachine
 - This is understandable, but incorrect, comparison. The biggest difference is that **virtual machines** virtualize hardware whereas **containers** virtualize operating system kernels.
 - Virtual machines run on a platform called a hypervisor. The hypervisor's main job is to translate operations on emulated hardware within virtual machines like memory processors, disks, etc, to operations on real hardware within their hosts.
 - Containers run on container run times. Container run times work with the operating system to allocate hardware and copy files and directories, including the parts with your application in it into something that looks more like any other app running on that system.
 -
## Docker Anatomy 
- A container is composed of two things: a Linux namespace and a Linux control group.
- Namespaces are a Linux kernel feature that provides the ability to expose different "views" of your system to applications running within it.
- Today's Linux kernels provide eight namespaces
- Control groups, another Linux kernel feature, build on this by providing the ability to restrict how much hardware each process can use. Docker uses control groups for a few things.
