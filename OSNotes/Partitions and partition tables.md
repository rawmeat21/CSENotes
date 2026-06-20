## What is a partition table?

The partitioning scheme is stored in a [partition table](https://wiki.archlinux.org/title/Partitioning#Partition_table).


Partition tables are MBR and GPT

### MBR

-> The Master Boot Record (MBR) is a special type of boot sector located at the very beginning of partitioned storage devices, such as hard drives and SSDs. 

-> It is a 512-byte data structure that serves as the first point of contact for the [BIOS](https://superops.com/blog/what-is-bios "https://superops.com/blog/what-is-bios") after the computer completes its initial power-on checks.

-> When you press the power button, BIOS loads the MBR into RAM and executes its code.

-> Through a process called **chain loading**, the MBR locates the partition containing the operating system files and hands over control to that partition’s Volume Boot Record (VBR). This ensures that the OS is loaded correctly and your system starts smoothly.

-> The MBR performs three distinct functions to ensure a successful boot:

- **Bootstrapping:** It contains the initial executable code required to facilitate the loading of the operating system's kernel.
    
- **Partitioning**: It holds the Master Partition Table, a database that tells the computer how the hard drive is divided (e.g., C: drive vs. D: drive) and which partition is marked as "active" or bootable.

- **Identification**: It includes a unique 32-bit disk signature that allows the operating system to identify the specific hard disk drive within the system, preventing conflicts if multiple drives are installed

![[Pasted image 20260620122207.png]]


### Components of MBR

The 512-byte structure of the Master Boot Record is precise and consists of three essential data structures.

- **Master Boot Code (Bootstrap Code)** – First 446 bytes, executable code that scans the partition table for the active partition. Corruption can cause startup errors like _“Error loading operating system.”_. Primary bootloader that searches the partition table, locates the active partition, and loads the secondary bootloader (like GRUB for Linux or Windows Boot Manager)
    
- **Disk Partition Table (DPT)** – Next 64 bytes, contains four entries describing partition size, type, and location. Limits MBR disks to four primary partitions.
    
- **Disk signature (Magic number)** – Last 2 bytes, always 0xAA55, acts as a boot validation check. BIOS skips the disk if this signature is missing.

Source: https://superops.com/tech-hub/what-is-mbr


In the MBR partition table (also known as DOS or MS-DOS partition table) there are 3 types of partitions:

- Primary
- Extended
    - Logical

**Primary** partitions can be bootable and are limited to 4 partitions per disk. 

If the MBR partition table requires more than four partitions, then one of the primary partitions needs to be replaced by an **extended** partition containing **logical** partitions within it.

Extended partitions can be thought of as containers for logical partitions. A hard disk can contain no more than 1 extended partition. The extended partition is also counted as a primary partition so if the disk has an extended partition, only three additional primary partitions are possible (i.e. three primary partitions and one extended partition). The number of logical partitions residing in an extended partition is unlimited. 

**A system that dual boots with Windows will require for Windows to reside in a primary partition.

Source: https://wiki.archlinux.org/title/Partitioning#Partition_table

### GUID Partition Table

-> The GUID Partition Table (GPT) is a modern partitioning scheme that replaces the older Master Boot Record (MBR) system.

-> The acronym “GUID” comes from its architecture; this partitioning scheme uses universally unique identifiers (UUIDs), also known as globally unique identifiers (GUIDs), to identify partitions and partition types.

-> It is part of the Unified Extensible Firmware Interface (UEFI) standard and offers a more advanced way to structure and manage storage devices.

-> Unlike MBR, which is limited to a maximum disk size of 2 terabytes (TB) and supports only four primary partitions, GPT removes these limitations and enables larger storage capacities with multiple partitions. Additionally, GPT provides better data integrity and redundancy, making it a more reliable option for modern computing.

Know more: https://wiki.archlinux.org/title/Partitioning#Partition_table

