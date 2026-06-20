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


