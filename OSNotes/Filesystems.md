In [computing](https://en.wikipedia.org/wiki/Computing), a **file system** or **file system** is used to control how data is [stored](https://en.wikipedia.org/wiki/Computer_data_storage) and retrieved.


### Why use filesystems?

Without a file system, information placed in storage would be one large body of data with no way to tell where one piece of information stops and the next begins. By separating the data into pieces and giving each piece a name, the information is easily isolated and identified.

### IMPORTANT

-> Information in RAM is _byte-addressable_: even if you’re only trying to store a boolean (1 bit), you need to read an entire byte (8 bits) to retrieve that boolean from memory, and if you want to flip the boolean, you need to write the entire byte back to memory. 

-> Hard drives are divided into _sectors_ (often 512 bytes large), and are _sector-addressable_: you must read or write entire 512-byte sectors, even if you’re only interested in 32 bytes of information within a sector.

![[Pasted image 20260620115026.png]]

-> A sector is a physical spot on a formatted disk. 

-> When a disk is formatted, tracks are defined (concentric rings from inside to the outside of the disk platter. Each track is divided into a slice, which is a sector. On hard drives and floppies, each sector can hold 512 bytes of data.

![[Pasted image 20260620115427.png]]

A block is a group of sectors that the operating system can address. A block might be one sector, or it might be several sectors (2,4,8, or even 16). The bigger the drive, the more sectors that a block will hold.

### Why are there blocks? Why doesn't the operating system just point straight to the sectors? 

**Because there are limits to the number of blocks, or drive addresses, that an operating system can address.** By defining a block as several sectors, an OS can work with bigger hard drives without increasing the number of block addresses. 

For example, PC DOS (earlier versions at least) could only address 65,536 blocks (64K), and each block could could only be a single sector. Thus, the largest size a disk volume could be was 32mb (64K * 512K). (Earlier versions of the Mac OS had a 16mb volume limit for similar reasons). If you increase the size of a block to, say, 4K, that same version of DOS can now work with volumes as large as 256MB (64K addresses * 4K blocks).


### Partitions

-> A partition is a logical division of a hard disk drive that's treated as a separate unit by operating systems ([OSes](https://www.techtarget.com/whatis/definition/operating-system-OS)) and [file systems](https://www.techtarget.com/searchstorage/definition/file-system).

-> Each partition can be treated as a discrete hard drive, supporting different file systems. This is useful for organizing files, separating operating systems, and enhancing your computer's efficiency and performance.

-> The disk stores the information about the partitions’ locations and sizes in an area known as the [partition table](https://en.wikipedia.org/wiki/Partition_table) that the operating system reads before any other part of the disk. When a hard drive is installed in a computer, it must be partitioned before you can format and use it.


![[Pasted image 20260620120258.png]]


### Some types of FS

**1. FAT (File Allocation Table):** FAT is one of the oldest and most common file systems. Due to its simple and efficient structure, it is widely used in small portable storage devices.

-> Compatibility with different OS−Compatible with many different operating systems, including Windows, Mac OS, and Linux. This makes it easy to share files between different computers and devices.

-> Easy to implement− FAT is a relatively simple file system that is easy to implement on different types of storage devices. This makes it a popular choice for removable storage devices such as USB drives and SD cards.

-> Supports large disk sizes− FAT supports large disk sizes, with the FAT32 version capable of supporting disks up to 2TB in size. 

-> Reduced risk of data corruption− Uses a journaling mechanism to minimize the risk of data corruption due to power failures or other system crashes. 

**2. NTFS (New Technology File System):** **NTFS** is a more advanced file system that was introduced with Windows NT. 

-> It supports larger file sizes, better security features, and improved performance compared to FAT.

-> **NTFS** uses a master file table to keep track of files and directories, and it includes features such as file compression, encryption, and disk quotas. 

-> **NTFS** is the default file system for most modern versions of Windows.


**3. EXT4:** It is a file system used in Linux operating systems. It has similar features to **NTFS**, but it is more efficient.

-> **Large File and Volume Support:** EXT4 caters to extensive storage needs by supporting large files and volumes.

-> **Delayed Allocation:** Delayed allocation improves performance and disk integrity by delaying block allocation until data is written to the disk.

-> **Unlimited Subdirectories**: EXT4 provides flexibility in file organization by supporting an unlimited number of subdirectories in a single directory.

-> **Journal Checksums:** Journal checksums enhance reliability and slightly improve performance by preventing disk I/O delays during journaling.

-> **Multiblock Allocator:** The multiblock allocator improves performance by efficiently organizing files on disk.


Others are: HFS (Hierarchical File System), **APFS:** Apple’s next-generation file system

Read more: https://medium.com/@eyupilis/file-systems-and-their-role-in-operating-systems-158cbf782530


