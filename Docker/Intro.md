"Docker is a set of platform as a service (PaaS) products that use OS-level virtualization to deliver software in packages called containers.".

![[Pasted image 20260819234425.png]]

- Docker is a set of tools to deliver software in containers.
- Containers are packages of software.

Containers allow the developer to personally run the application inside a container, which then includes all of the dependencies required for the app to work.


Difference between VMs:

![[Pasted image 20260819235235.png]]

Virtual Machines (VMs) run on a [hypervisor (opens in a new tab)](https://en.wikipedia.org/wiki/Hypervisor), which virtualizes the physical hardware. Each VM includes a full operating system (OS) along with the necessary binaries and libraries, making them heavier and more resource-intensive. Containers, on the other hand, share the host OS kernel and only package the application and its dependencies, resulting in a more lightweight and efficient solution.

VMs provide strong isolation and are suited for running multiple OS environments, but they have a performance overhead and longer startup times. Containers offer faster startup, better resource utilization, and high portability across different environments, though their isolation is at the process level, which may not be as robust as that of VMs. Overall, VMs could be used for scenarios needing complete OS environments, while containers excel in lightweight, efficient, and consistent application deployment.

Docker relies on Linux kernels, which means that macOS and Windows cannot run Docker natively without some additional steps. Each operating system has its own solution for running Docker. For example, Docker for Mac actually uses a Linux virtual machine under the hood, within which Docker operates.

