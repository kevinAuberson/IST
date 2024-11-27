# Lab 01: File systems
_Duration of this lab: 4 periods_

#### PEDAGOGICAL OBJECTIVES

- Prepare (partition and format) an external backup disk from scratch
    
- Become familiar with block devices
    
- Repair a corrupted file system
    
- Create a file system in a file
    

#### TASKS

In this lab you will perform a number of tasks and document your progress in a lab report (PDF file). Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.

#### PREREQUISITES

This lab requires an installation of Ubuntu 22.04 LTS on a physical or virtual machine. If you followed the ADS course last semester you can reuse the virtual machine.

### TASK 1: EXPLORE BLOCK DEVICES AND FILESYSTEMS

In this task you will explore how Linux handles block devices and filesystems.

1. Using the `lsblk` command, list the existing block devices.
    
    - On which block device is the boot partition mounted? In the `/dev` directory find the special file corresponding to this block device. With `ls -l` list its metadata.
    - On which block device is the root (/) partition mounted? What is the name of its special file?
    - With `hdparm -t` do a timing test on the boot partition. What throughput do you get?
2. Convince yourself that the special file representing the root (/) partition can be read just like any other file. As it contains binary data, just opening it with `less` will mess up the terminal, so use the `xxd` hexdump utility.
    
    - To see how `xxd` works, create a small text file and open it with `xxd -a`.
    - Now open the special file with the same command. You may pipe its output into `less`. What do you see? If your root partition uses LVM (verify with `lsblk`), you should see text strings containing volume group configuration information.
3. As the special file represents all the blocks of a partition, the content of all files of the root partition should be there. Pick a text file at random (for example a file in `/usr/share/doc/`) and try to find its content in the special file.
    
    - Tip: There is a filter utility that looks for data that looks like text strings and removes everything else: the `strings` command.

### TASK 2: PREPARE AND PARTITION A DISK

In this task you will use your Linux VM to prepare an external disk. It is recommended you use a USB stick that you can erase completely. For the purposes of this lab the USB stick behaves like any external disk, and we will call it a disk in the following.

If you do not have a USB stick available use your virtualization software to create a small virtual disk (30 MB should be enough) and attach it to the Linux VM. You may have to shut down the VM before you can add a new disk. Do that only after you have done the first step below.

1. Before you plug in the disk, list the existing block devices. Using the `findmnt` command find all the partitions that are already mounted.
    
    - In `findmnt`'s output you will see many pseudo file systems. You can suppress them with the `--real` option.
2. Attach the disk to your computer.
    
    List again the block devices. Which new block devices and special files appeared? These represent the disk and its partitions you just attached.
    
3. Create a partition table on the disk and create two partitions of equal size using the `parted` tool.
    
    You can consult [Gentoo Linux Documentation -- Preparing the Disks](https://www.gentoo.org/doc/en/handbook/handbook-amd64.xml?part=1&chap=4) as a reference on how to use `parted`.
    
    - Using superuser privileges invoke `parted` with a single parameter which is the special file representing the disk. Be careful not to confuse the special file for the disk (ending in a letter) and for the partitions (ending in a number).
        
    - Display the existing partitions with the `print` command. If the disk is completely blank you will get an error message about a missing disk label.
        
    - Use the `mktable` command to create a partition table (overwriting any existing one). It should have Master Boot Record (MBR) layout (i.e. label type `msdos`).
        
    - Display the free space with the command `print free` (roughly the size of the disk minus some overhead). Write the value down.
        
    - Use the `mkpart` command to create the partitions.
        
        - The first partition will
            - be a primary partition
            - have a file system type of `fat32`
            - start at 0
            - end at about half the free space.
        - The second partition will
            - be a primary partition
            - have a file system type of `ext4`
            - start at half the free space
            - end at the free space.
    - Quit parted and verify that there are now two special files in `/dev` that correspond to the two partitions.
        
4. Format the two partitions using the `mkfs` command.
    
    - The first partition should have the file system type `vfat`.
    - The second partition should have the file system type `ext4`.
5. Create two empty directories in the `/mnt` directory as mount points, called `part1` and `part2`. Mount the newly created file systems in these directories.
    
6. How much free space is available on these filesystems? Use the `df` command to find out. What does the `-h` option do?
    

### TASK 3: EXPLORE THE FILE SYSTEM SUPPORT IN THE KERNEL

In this task you will explore the parts of the kernel that implement the different file systems.

1. Find out which file systems the kernel supports right now. The kernel makes information about itself available to userspace programs in a pseudo file system that is mounted at `/proc`. The files in that file system describe kernel objects.
    
    - List the content of `/proc`. What is the version of the kernel in `/proc/version`?
    - The directories with numbers represent the running processes. The numbers are the process ids. Display the process id of your bash session with `echo $$`. List the information in the corresponding directory. What was the command line that started this process (look in `cmdline`)?
    - The kernel lists the file systems it supports right now file `filesystems`. List them.
    - Can you find the `proc` filesystem itself in the list? How is it tagged? All file systems with that tag are pseudo file systems.
    - List the real (non-pseudo) file systems.
2. Find out which file systems the kernel is able to support by looking at the available kernel modules. The files containing kernel modules can be found at `lib/modules/<kernel version>/kernel/fs`. List them.
    
    As you see there are more file systems there. They can be dynamically loaded by the kernel by loading the module, e.g., when you insert a new disk.
    
3. When a new disk is inserted the kernel knows which file system to activate by looking at a label that indicates the type of file system. That label is part of the partition metadata (called _signature_). Use the `blkid` command to list the metadata of all known partitions (mounted or not). Note that you might need to run the command with admin permissions to display all partitions metadata.
    
    - Verify that the partitions you created are labeled correctly.
    - There is another piece of information in the partition metadata. What does it do?
4. An older way for the kernel to find out which file system to activate is the file `/etc/fstab`. This file lists all the file systems that should be mounted when the system boots. It indicates the special file that represents the partition, the directory where it should be mounted (the _mount point_), and the file system to activate.
    
    - List the content of `/etc/fstab`. What line is responsible for mounting the root (/) file system? This line has a particular way of referencing the partition, how?

### TASK 4: MANAGE AN EXT4 PARTITION

In this task you will test the integrity of an ext4 partition and repair it when it is damaged.

1. Unmount the ext4 partition on the external disk.
    
2. Run a file system check using the `fsck` command.
    
3. Add the `-f` option to the command to force a complete verification.
    
4. Display the file system structure with the `dumpe2fs` command. How many inodes are unused?
    
5. Intentionally corrupt the file system by overwriting 4 MB of data, starting 10 kB in:
    
    ```
     sudo dd if=/dev/zero of=<block device> bs=1k seek=10 count=4k
    ```
    
    **Caution! Be careful to specify the correct block device.f**
    
6. Try to mount the partition. You should get an error message. Repair the file system with the `fsck` command.
    
7. Mount the repaired partition.
    

### TASK 5: CREATE A FILE SYSTEM IN A FILE

In this task you will create a file system in a file instead of a partition. For that we use a so-called loopback device. A loopback device is a device that can be used as a block device to access files. The loopback device appears as another special file in `/dev`.

1. Create a 100 MB file using `dd`:
    
    ```
     dd if=/dev/zero of=/tmp/bigfile bs=1M count=100
    ```
    
2. Find the next available loopback device:
    
    ```
     losetup -f
    ```
    
    For the following, let's say `losetup -f` returned `/dev/loop6`.
    
3. Associate the loopback device with the file:
    
    ```
     sudo losetup /dev/loop6 /tmp/bigfile
    ```
    
4. Verify that the association is OK:
    
    ```
     losetup -a 
    ```
    
5. Create an `ext4` file system on block device `/dev/loop6`. Create a mountpoint in `/mnt/bigfile`. Mount the file system on the mountpoint. How does `findmnt` show the new file system?
    
6. Create a few files in the file system with unique strings. By searching the content of `bigfile`, can you find the strings? Use the `sync` command to force the kernel to write buffered data to disk.
    
7. Undo everything:
    
    - Unmount the file system
    - Free the loopback device with `losetup -d /dev/loop6` and verify with `losetup -a`.

Note: There is a shortcut in the `mount` command to allocate the loopback device on the fly by using the `-o loop` option, which makes dealing with `losetup` unnecessary. The intention here was to show the inner workings.
