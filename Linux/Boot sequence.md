![[Pasted image 20260819102047.png]]

Before the system is fully booted, filesystems must be checked and mounted and
system daemons started. These procedures are managed by a series of shell scripts
(sometimes called “init scripts”) or unit files that are run in sequence by init or
parsed by systemd.

### System firmware

When a machine is powered on, the CPU is hardwired to execute boot code stored
in ROM. On virtualized systems, this “ROM” may be imaginary, but the concept
remains the same.

The system firmware typically knows about all the devices that live on the mother-
board, such as SATA controllers, network interfaces, USB controllers, and sensors
for power and temperature. 

In addition to allowing hardware-level configuration of these devices, the firmware lets you either expose them to the operating system or disable and hide them.

During normal bootstrapping, the system firmware probes for hardware and disks, runs a simple set of health checks, and then looks for the next stage of bootstrapping code. The firmware UI lets you designate a boot device, usually by prioritizing a list of available options (e.g., “try to boot from the DVD drive, then a USB drive, then a hard disk”).