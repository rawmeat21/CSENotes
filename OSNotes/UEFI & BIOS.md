![[Pasted image 20260620125103.png]]


When you press the power button, the CPU begins execution at a predefined reset vector, which is mapped to the system firmware (UEFI/ BIOS) stored in non-volatile memory. After this, the process looks something like this:

1. **POST:** The firmware runs POST (Power On Self Test), a diagnostic check, to ensure all critical hardware components like CPU, memory, storage drives, and input/output devices are working properly. And if any issues are found, they are usually signaled with beeps or error codes, and the boot process is paused. 

2. **Device Init:** When the POST is successful, the firmware initializes the peripheral devices, so they are ready to function as needed. 

3. **Finding the bootloader:** Next, the firmware code looks at the configured boot order to find a storage device that contains a valid bootloader. **The primary job of this bootloader is to find the operating system.**

4. Once the boot device is identified, the firmware reads the first sector, which contains the bootloader program. Boot-loader is only available in the first sector of a disk, which is 512 bytes.

5. Once the bootloader is located, the firmware loads it into memory and hands over control of the system to it. The bootloader, which provides a more advanced environment, loads the **OS kernel** into the memory and then transfers control to it. Unix-like operating systems then run the `init` process. 

6. The OS then takes over to finish the boot process. It will initialize system components and file systems, load drivers, and prepare the user interface or login screen so the user can begin working.

**GRUB is the standard bootloader for UNIX-like operating systems, such as LINUX. It can also load Windows, but Windows typically uses its own proprietary bootloader, called Windows Boot Manager.**


### Legacy BIOS

-> The legacy BIOS is the original firmware interface that acts as a fundamental link between your computer's hardware and its operating system. It initializes the [hardware](https://superops.com/blog/managed-service-provider/driving-the-hardware-replacement-cycle-blog "/blog/managed-service-provider/driving-the-hardware-replacement-cycle-blog") and starts the above-mentioned boot process.

-> Historically stored in a small ROM chip on the motherboard, which later changed to EPROM, EEPROM, and flash memory, supporting easier updates and bug fixes.

-> Can only handle boot disks up to 2.2 TB because it relies on the older MBR partitioning scheme. Plus, its setup screen is text-based, navigated with the keyboard, and offers only a handful of configuration options.

![[Pasted image 20260620125537.png]]

![[Pasted image 20260819103139.png]]


Source: https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface

The [Unified Extensible Firmware Interface](https://en.wikipedia.org/wiki/UEFI "wikipedia:UEFI") **(UEFI)** is an interface between operating systems and firmware. It provides a standard environment for booting an operating system and running pre-boot applications.


### UEFI

-> Unlike BIOS, which relies on the MBR in the first sector of a disk, UEFI uses the EFI System Partition (ESP), a dedicated disk partition containing .efi executables, including bootloaders.

-> It stores all data about initialization and startup in an .efi file, which is located on a specific partition called the EFI System Partition (ESP) on the hard disk. This ESP partition also contains the bootloader.

![[Pasted image 20260620130317.png]]

-> UEFI utilizes the GUID Partition Table (GPT). GPT removes the 2-terabyte limit, allowing support for modern, high-capacity hard drives and solid-state drives.

**Advantages**:

1. UEFI supports drive sizes upto 9 zettabytes, whereas BIOS only supports 2.2 terabytes.
2. UEFI provides faster boot time.
3. UEFI has discrete driver support, while BIOS has drive support stored in its ROM, so updating BIOS firmware is a bit difficult.
4. UEFI offers security like "Secure Boot", which prevents the computer from booting from unauthorized/unsigned applications. 
5. UEFI runs in 32bit or 64bit mode, whereas BIOS runs in 16bit mode. So UEFI is able to provide a GUI (navigation with mouse) as opposed to BIOS which allows navigation only using the keyboard.

Source: https://superops.com/blog/bios-vs-uefi, https://www.freecodecamp.org/news/uefi-vs-bios/

![[Pasted image 20260819103641.png]]
![[Pasted image 20260819103652.png]]
![[Pasted image 20260819103715.png]]
![[Pasted image 20260819103723.png]]


Example of my system:

```bash
❯ efibootmgr -v
BootCurrent: 0003
Timeout: 0 seconds
BootOrder: 0003,0000,0001,0002,0004,9999
Boot0000* debian	HD(1,GPT,ad7b5b1b-90b7-4573-865e-0e0d04264f5e,0x800,0x82000)/\EFI\debian\shimx64.efi
      dp: 04 01 2a 00 01 00 00 00 00 08 00 00 00 00 00 00 00 20 08 00 00 00 00 00 1b 5b 7b ad b7 90 73 45 86 5e 0e 0d 04 26 4f 5e 02 02 / 04 04 34 00 5c 00 45 00 46 00 49 00 5c 00 64 00 65 00 62 00 69 00 61 00 6e 00 5c 00 73 00 68 00 69 00 6d 00 78 00 36 00 34 00 2e 00 65 00 66 00 69 00 00 00 / 7f ff 04 00
Boot0001* Windows Boot Manager	HD(1,GPT,ad7b5b1b-90b7-4573-865e-0e0d04264f5e,0x800,0x82000)/\EFI\Microsoft\Boot\bootmgfw.efi57494e444f5753000100000088000000780000004200430044004f0042004a004500430054003d007b00390064006500610038003600320063002d0035006300640064002d0034006500370030002d0061006300630031002d006600330032006200330034003400640034003700390035007d00000000000100000010000000040000007fff0400
      dp: 04 01 2a 00 01 00 00 00 00 08 00 00 00 00 00 00 00 20 08 00 00 00 00 00 1b 5b 7b ad b7 90 73 45 86 5e 0e 0d 04 26 4f 5e 02 02 / 04 04 46 00 5c 00 45 00 46 00 49 00 5c 00 4d 00 69 00 63 00 72 00 6f 00 73 00 6f 00 66 00 74 00 5c 00 42 00 6f 00 6f 00 74 00 5c 00 62 00 6f 00 6f 00 74 00 6d 00 67 00 66 00 77 00 2e 00 65 00 66 00 69 00 00 00 / 7f ff 04 00
    data: 57 49 4e 44 4f 57 53 00 01 00 00 00 88 00 00 00 78 00 00 00 42 00 43 00 44 00 4f 00 42 00 4a 00 45 00 43 00 54 00 3d 00 7b 00 39 00 64 00 65 00 61 00 38 00 36 00 32 00 63 00 2d 00 35 00 63 00 64 00 64 00 2d 00 34 00 65 00 37 00 30 00 2d 00 61 00 63 00 63 00 31 00 2d 00 66 00 33 00 32 00 62 00 33 00 34 00 34 00 64 00 34 00 37 00 39 00 35 00 7d 00 00 00 00 00 01 00 00 00 10 00 00 00 04 00 00 00 7f ff 04 00
Boot0002* Internal Hard Disk	PciRoot(0x0)/Pci(0x2,0x4)/Pci(0x0,0x0)/NVMe(0x1,00-A0-75-01-48-44-39-96)/HD(1,GPT,ad7b5b1b-90b7-4573-865e-0e0d04264f5e,0x800,0x82000)0000424f
      dp: 02 01 0c 00 d0 41 03 0a 00 00 00 00 / 01 01 06 00 04 02 / 01 01 06 00 00 00 / 03 17 10 00 01 00 00 00 00 a0 75 01 48 44 39 96 / 04 01 2a 00 01 00 00 00 00 08 00 00 00 00 00 00 00 20 08 00 00 00 00 00 1b 5b 7b ad b7 90 73 45 86 5e 0e 0d 04 26 4f 5e 02 02 / 7f ff 04 00
    data: 00 00 42 4f
Boot0003* GRUBB	HD(7,GPT,7019f61d-0625-405f-b8cd-f4089633110e,0x38c18800,0x200000)/\EFI\GRUBB\grubx64.efi
      dp: 04 01 2a 00 07 00 00 00 00 88 c1 38 00 00 00 00 00 00 20 00 00 00 00 00 1d f6 19 70 25 06 5f 40 b8 cd f4 08 96 33 11 0e 02 02 / 04 04 32 00 5c 00 45 00 46 00 49 00 5c 00 47 00 52 00 55 00 42 00 42 00 5c 00 67 00 72 00 75 00 62 00 78 00 36 00 34 00 2e 00 65 00 66 00 69 00 00 00 / 7f ff 04 00
Boot0004* Internal Hard Disk	PciRoot(0x0)/Pci(0x2,0x4)/Pci(0x0,0x0)/NVMe(0x1,00-A0-75-01-48-44-39-96)/HD(7,GPT,7019f61d-0625-405f-b8cd-f4089633110e,0x38c18800,0x200000)0000424f
      dp: 02 01 0c 00 d0 41 03 0a 00 00 00 00 / 01 01 06 00 04 02 / 01 01 06 00 00 00 / 03 17 10 00 01 00 00 00 00 a0 75 01 48 44 39 96 / 04 01 2a 00 07 00 00 00 00 88 c1 38 00 00 00 00 00 00 20 00 00 00 00 00 1d f6 19 70 25 06 5f 40 b8 cd f4 08 96 33 11 0e 02 02 / 7f ff 04 00
    data: 00 00 42 4f
Boot9999* USB Drive (UEFI)	PciRoot(0x0)/Pci(0x1d,0x0)/USB(16,0)0000424f
      dp: 02 01 0c 00 d0 41 03 0a 00 00 00 00 / 01 01 06 00 00 1d / 03 05 06 00 10 00 / 7f ff 04 00
    data: 00 00 42 4f
```

```
Boot0003* GRUBB	HD(7,GPT,7019f61d-0625-405f-b8cd-f4089633110e,0x38c18800,0x200000)/\EFI\GRUBB\grubx64.efi
```

| Parameter Field          | Value in `Boot0003`                    | Technical Meaning                                                                                             |
| ------------------------ | -------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Partition Index**      | `7`                                    | The partition table entry index (Partition 7).This is how your UEFI knows where to look.                      |
| **Partition Table Type** | `GPT`                                  | Specifies that the disk uses the GUID Partition Table scheme (not legacy MBR).                                |
| **PARTUUID**             | `7019f61d-0625-405f-b8cd-f4089633110e` | The globally unique identifier (128-bit UUID) assigned to Partition 7 in the GPT partition header.            |
| **Start LBA**            | `0x38c18800`                           | The exact starting sector (Logical Block Address) on the physical NVMe drive in hex (952,203,264 in decimal). |
| **Size LBA**             | `0x200000`                             | The exact partition length in sectors (2,097,152 sectors ×512 bytes=1 GiB).                                   |

```bash
❯ lsblk -o NAME,FSTYPE,MOUNTPOINTS,PARTTYPE,SIZE
NAME        FSTYPE MOUNTPOINTS      PARTTYPE                               SIZE
nvme0n1                                                                  476.9G
├─nvme0n1p1 vfat                    c12a7328-f81f-11d2-ba4b-00a0c93ec93b   260M
├─nvme0n1p2                         e3c9e316-0b5c-4db8-817d-f92df00215ae    16M
├─nvme0n1p3 ntfs                    ebd0a0a2-b9e5-4433-87c0-68b6b72699c7 200.6G
├─nvme0n1p4 ext4   /mnt/data        ebd0a0a2-b9e5-4433-87c0-68b6b72699c7   136G
├─nvme0n1p5 ext4   /                ebd0a0a2-b9e5-4433-87c0-68b6b72699c7 107.4G
├─nvme0n1p6 swap   [SWAP]           ebd0a0a2-b9e5-4433-87c0-68b6b72699c7   9.8G
├─nvme0n1p7 vfat   /boot            c12a7328-f81f-11d2-ba4b-00a0c93ec93b     1G
├─nvme0n1p8 ntfs   /mnt/SharedDrive ebd0a0a2-b9e5-4433-87c0-68b6b72699c7  21.1G
└─nvme0n1p9 ntfs                    de94bba4-06d1-4d40-a16a-bfd50179d6ac   795M
```

```
nvme0n1p7 vfat   /boot            c12a7328-f81f-11d2-ba4b-00a0c93ec93b     1G
```
This is where the UEFI looks for my bootloader.

It checks this path: `\EFI\GRUBB\grubx64.efi`. You will find this path under `/boot`, because the kernel actually mounts partition 7 to `/boot` (see lsblk output).

See the partitions not mounted? This is part of Windows. Windows itself is on the big 200.6G partition (p3). My arch is on partition 5. 

`grubx64.efi` is my bootloader. 

```
[ Power-On ]
     │
     ▼
1. Fetch Variable 'Boot0003' from NVRAM
     │
     ▼
2. Parse PCI/NVMe Controller Route
   └── PciRoot(0x0)/Pci(0x2,0x4)/NVMe(...)
     │
     ▼
3. Read Primary GPT Partition Table from Disk
     │
     ▼
4. Locate Partition Entry
   ├── Match Index: 7
   ├── Match PARTUUID: 7019f61d-0625-405f-b8cd-f4089633110e
   └── Match Start Sector: 0x38c18800
     │
     ▼
5. Initialize FAT Driver on Sector Range [0x38c18800 ... 0x38E18800]
     │
     ▼
6. Traverse FAT32 Root Directory ──► \EFI\GRUBB\grubx64.efi
```

Once UEFI loads `grubx64.efi` into RAM and transfers CPU control to it, GRUB takes over execution:

- **Reads Configuration:** GRUB loads `/boot/grub/grub.cfg` from Partition 7 (`/dev/nvme0n1p7`).
    
- **Renders Menu / Timeout:** GRUB displays the boot menu (or immediately executes the default entry if the timeout is `0`).
    
- **Loads Artifacts into RAM:** Based on the entry, GRUB allocates system RAM and loads:
    
    - The Linux kernel binary (`/boot/vmlinuz-linux`)
        
    - The initramfs image (`/boot/initramfs-linux.img`)

- **Handoff to Kernel:** GRUB passes kernel parameters (like `root=UUID=... rw`) and jumps the CPU execution pointer to the starting memory address of `vmlinuz-linux`. At this exact instant, GRUB is wiped from RAM, and the Linux kernel takes control.


```
/boot🔒 
❯ ls   
 EFI   grub   initramfs-linux.img  'System Volume Information'	vmlinuz-linux
```

`vmlinuz-linux` is basically the linux kernel.

- **`vm`**: Stands for **Virtual Memory**. This prefix originated when UNIX/Linux kernels added virtual memory paging support (differentiating them from older flat-memory operating systems).
    
- **`linu`**: Short for **Linux**.
    
- **`z`**: Denotes **compression**. An uncompressed kernel binary produced after compilation is an ELF file named `vmlinux`. Compressing `vmlinux` using `zstd`, `gzip`, or `xz` produces `vmlinuz`.

