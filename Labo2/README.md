# Labo 2: Logical Volumes and snapshots
_Duration of this lab: 4 periods_

#### PEDAGOGICAL OBJECTIVES

The developers of traditional file systems designed them for disks that are installed inside a computer. They assumed that a disk cannot change in size and that you cannot add a disk while the computer is running. Then came external disks that you plug in while the computer is running, and later remote disks via a SAN or via cloud computing.

To make file systems more flexible, the designers of the Linux kernel added a layer between the kernel's file systems and the disks (block devices) called Logical Volume Manager. This layer provides virtual disks, called Logical Volumes, that aggregate several disks and that can grow and shrink in size.

Recent file systems (ZFS, Btrfs, APFS) include the functionality of Logical Volume Manager directly in the file system, but their use is not yet widespread.

In this lab you will:

- Become familiar with Logical Volumes and Logical Volume Manager.
    
- Extend the capacity of a file system on the fly by adding a Physical Volume to its underlying Volume Group.
    
- Create a snapshot of a file system by snapshotting its Logical Volume.
    

#### TASKS

In this lab you will perform a number of tasks and document your progress in a lab report (PDF file). Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.

#### PREREQUISITES

This lab requires

- an installation of Ubuntu 22.04 LTS on a physical or virtual machine.
    
- an external disk of at least 100 MB capacity (physical or virtual).
    

## TASK 1: CREATE PHYSICAL VOLUMES

In this task you will reset your external disk and configure Logical Volume Manager (LVM) Physical Volumes (PV) on it, so you can later create Logical Volumes on it.

1. Reset your external disk. Using `parted` remove all partitions, or simply write a new partition table.
    
2. Create four partitions with these characteristics: primary, 25 MB size, type `ext4`.
    
3. List the available LVM commands. They belong to the Debian package _lvm2_ which should already be installed. Use `dpkg` with the `-L` option to list the content of the package. The commands are all located in the `/sbin` directory. Use `grep` to filter and `sort` to sort alphabetically.
    
4. List all partitions that could potentially host a Physical Volume by using `pvs` with the `--all` option.
    
5. On each of the four partitions of your external disk, create a Physical Volume using `pvcreate`. Add the `-vv` option so that it tells you in detail what it is doing. For the first partition copy the output of the command into the report, but copy only the lines about the partition that receives the Physical Volume and ignore the other messages.
    
6. Display detailed information about the first Physical Volume using `pvdisplay`.
    

## TASK 2: CREATE TWO VOLUME GROUPS

In this task you will create Volume Groups that combine several Physical Volumes into a bigger storage space.

1. Create a first Volume Group `lab-vg1` that contains only the first Physical Volume. Display the Physical Volume again with `pvdisplay`. What has changed?
    
2. Create a second Volume Group `lab-vg2` that contains Physical Volumes 2 and 3.
    
3. List all Volume Groups with `vgs`. Then list all Physical Volumes with `pvs`. What do you see?
    

## TASK 3: CREATE LOGICAL VOLUMES

In this task you will create Logical Volumes that will contain file systems.

1. On the Volume Group `lab-vg1` create a Logical Volume of size 20 MB with the command `lvcreate -L 20M lab-vg1`.
    
2. Verify hat the new volume appears when you use `lvs` to list Logical Volumes. Also verify that it appears when you use `lsblk` to list the block devices. What is the name of the special file in `/dev` that represents the volume?
    
3. Create an ext4 file system on the volume. Mount the volume. Fill the file system with a 14 MB file using `dd` (Google it).
    
4. On the Volume Group `lab-vg2` create another Logical Volume of size 20 MB, create an ext4 file system on it and mount it. Create a file named `foo` that contains the text `111`.
    

## TASK 4: GROW A FILE SYSTEM WHILE IT IS IN USE

In the previous task you filled a Logical Volume's file system completely. The Volume Group does not have any free space left either. In this task you will extend the Volume Group with the Physical Volume that's still unused to make more space. Then you use that space to extend the Logical Volume. Finally you grow the file system to make use of the additional space in the Logical Volume. The file system ext4 supports growing while it is being mounted. That is a big advantage, as applications can continue to run without interruption.

1. Verify that the file system is indeed full (use `df -h`).
    
2. Verify that the Volume Group is full (use `vgs`).
    
3. Extend the Volume group using `vgextend` and verify with `vgs`.
    
4. Extend the Logical Volume by an additional 20 MB using `lvextend --size <new_size> <volume_group>/<logical_volume>`.
    
5. Grow the file system while it is mounted using `resize2fs` and verify its new capacity with `df -h`. Note: not all file systems support growing while being mounted. In that case you have to stop all applications using the file system, unmount, grow, remount, and restart the applications.
    

## TASK 5: CREATE A SNAPSHOT

In task 3 you created a file containing `111` on the file system on the second Logical Volume. In this task you create a snapshot of the volume which behaves like a copy.

1. Create a snapshot volume using the `--snapshot` option of `lvcreate`. Use the `--name` option to give it the name `snap`. You also need to specify a size for the snapshot with the `--size` option. Remember that initially a snapshot does not consume any storage blocks as the data in the original volume and the snapshot volume is identical. It is only when the data in the two volumes starts deviating that storage blocks are needed. The size of the snapshot determines how many data blocks can be different.
    
    To make things interesting, specify a size less than the size of the original volume.
    
    Note that LVM snapshots are read/write by default.
    
2. Display an overview of all Logical Volumes using `lvs`. Which column shows the name of the original volume?
    
3. Display the charactersticts of the snapshot volume using `lvdisplay`.
    
    - What line shows the name of the original volume?
    - What line shows the size of the original volume?
    - What line shows the space allocated for the snapshot volume?
    - What does COW stand for?
4. Mount the snapshot volume. Using the file `foo` you created earlier verify that the two volumes behave like independent copies.
    
5. Make the data of the original volume change completely by using the `dd` command to write a new file of 14 MB size. Run `df -h` to see how it affects the fullness of the original volume and the snapshot. What do you see?
    
    - The way that you allocated it, is the snapshot volume able support a change of 14 MB of data?
        
    - What happened? Why?
        
6. Remove the broken snapshot volume.
    
7. Redo the above, this time allocating sufficient space to the snapshot volume to support a complete change of data of the original volume.
    

### TASK 6: PROVISION A THIN VOLUME AND SNAPSHOT IT

The Logical Volumes we have created so far are so-called _thickly provisioned_. That means that if we create a Logical Volume of, say 20 MB capacity, LVM makes an allocation (reservation) of a sufficient number of blocks on the Physical Volumes to store 20 MB of data.

_Thinly provisioned_ Logical Volumes on the other hand do not have their blocks allocated when they are created. The volume is completely virtual. LVM monitors the write operations into the volume and allocates physical blocks only when needed. So you can for example create a 4 TB volume, write 40 MB of data into it, and the volume will only take up 40 MB of physical blocks.

LVM takes the physical space for thin volumes from a pool of blocks which is called a _thin pool_. It is called a pool because several thin volumes can share the same thin pool. Before you can create a thin volume you have to create a thin pool.

1. Remove all Logical Volumes from Volume Group `lab-vg2`.
    
2. Follow the explanations in the Ubuntu manual on [lvmthin](https://manpages.ubuntu.com/manpages/bionic/en/man7/lvmthin.7.html) to create
    
    - a thin data Logical Volume called `pool0` of 28 MB
    - a thin metadata Logical Volume called `pool0meta` of 4 MB
3. Combine the two into a thin pool Logical Volume. List the Logical Volumes using `lvs`. Use the `-a` option to list also the hidden ones.
    
4. Create a thin Logical Volume from the thin pool named `thin1` and give it a size of 80 MB, although the thin pool only has 28 MB capacity. What warnings to you see?
    
5. Create an ext4 file system on `thin1`. Mount the file system. How much capacity does `df -h` see in the file system?
    
6. Do experiments: Fill the file system with a bit of data by using `dd` to write files and verify that it behaves normally. Then write more and more data until you cross the size of the thin pool and see what happens. You can see LVM's log messages by using the `dmesg` command, they appear as `device-mapper`. What do you observe?
    

### TASK 7: SCENARIO

You are a data engineer in a company. You need to set up the backup system for your production system, which runs a large database. Backups are important in case of unexpected incidents or human error. As the volume of data is significant, the backup process's duration can be fairly long, typically between 30 minutes and 1 hour.

Design a solution for this backup system using snapshots, knowing that:  

* The backup should be performed without interrupting the database operations.  

* The backup files need to be physically distant and sent to another datacenter located a few kilometers away.

Describe a potential solution and explain your thought process.