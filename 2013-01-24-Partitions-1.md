<!-- vim:fo=tq:tw=144:syntax=off:sw=2:ts=2: -->

Partitions - A Thing Of The Past (Part 1)
=========================================

##TL;DR##

Virtualization technologies like VMWare Workstation or vSphere remove the need for MBR or other partitioning schemes for Linux guest operating
system disks. By using the full disk when creating a file system (`mkfs -t ext4 /dev/sdb`, for instance), you gain the ability to expand any
drive, even the live `/` file system, at runtime.

Part 1 examines expanding a Linux root file system the old fashioned way, using tools like `parted`, `fdisk` and a reboot.

##The Full Story##

Recently I began investigating options to support resizing virtual disks used by Linux guest operating systems within vSphere 5. Ideally, online
expansion of any file system would be supported without rebooting the system. I quickly found out, however, that modifications of the partition
table would not be read by the kernel, even after executing `partprobe`. Here's an example session using Ubuntu 10 demonstrating the resize of
the root file system within vSphere - I expanded the VMs VMDK file to 26GiB using vSphere Client but did not reboot the VM.

This command prints out the current partition scheme. The default Ubuntu setup created a primary partition (#1) for the `/` file system and boot
loader, and an extended partition (#2) which contains a logical partition (#5) for swap space:

    root@ubuntu:~# parted -l
    Model: VMware, VMware Virtual S (scsi)
    Disk /dev/sda: 21.5GB
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos

    Number  Start   End     Size    Type      File system     Flags
    1      1049kB  20.5GB  20.5GB  primary   ext4            boot
    2      20.5GB  21.5GB  938MB   extended
    5      20.5GB  21.5GB  938MB   logical   linux-swap(v1)

This command iterates over all of the SCSI devices and forces them to rescan:

    root@ubuntu:~# for scsi_device in /sys/class/scsi_device/*; do echo 1 > $scsi_device/device/rescan; done

After the rescan, you can see that `/dev/sda` reports a larger size (`/dev/sda:26.0GiB`):

    root@ubuntu:~# parted -ms /dev/sda 'u GiB p'
    BYT;
    /dev/sda:26.0GiB:scsi:512:512:msdos:VMware, VMware Virtual S;
    1:0.00GiB:19.1GiB:19.1GiB:ext4::boot;
    2:19.1GiB:20.0GiB:0.87GiB:::;
    5:19.1GiB:20.0GiB:0.87GiB:linux-swap(v1)::;

Continuing on with the partition expansion, there's no need to have swap on an extended partition, so we'll delete partitions #2 and #5, expand #1,
and recreate swap on its own primary partition. Since `parted` won't touch an in-use file system, we turn swap off and have to resort to
`fdisk` to resize partition #1.

NOTE: when recreating the boot partition, you must start at _exactly the same sector_ as the original partition!

This command will show the current partition layout in sectors, note that partition #1 starts at sector 2048:

    root@ubuntu:~# parted -s /dev/sda 'u s p'
    Model: VMware, VMware Virtual S (scsi)
    Disk /dev/sda: 41943040s
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos

    Number  Start      End        Size       Type      File system     Flags
    1      2048s      40105983s  40103936s  primary   ext4            boot
    2      40108030s  41940991s  1832962s   extended
    5      40108032s  41940991s  1832960s   logical   linux-swap(v1)

These commands do the following:

* `swapoff -a` turns the swap off to allow deletion of partitions #5 and #2
* The `parted` commands do the following:
    * `rm 5` deletes partition #5
    * `rm 2` deletes partition #2
    * `rm 1` attempts to delete partition #1, but fails

Terminal session:

    root@ubuntu:~# swapoff -a
    root@ubuntu:~# parted
    GNU Parted 2.2
    Using /dev/sda
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) rm 5
    (parted) rm 2
    (parted) rm 1
    Error: Partition /dev/sda1 is being used. You must unmount it before you modify it with Parted.

Since `parted` won't modify partition #1 for us, we must switch to using the more unfriendly `fdisk` command. The following `fdisk` commands do the following:

* `u` switches to sectors
* `d` deletes partition #1
* `n` create a new partition:
    * `p` create primary partition
    * `1` creating partition #1
    * `2048` start at sector 2048 (the original value)
    * `+24G` end 24GiB later
* `n` create a new partition:
    * `p` create primary partition
    * `2` creating partition #2
    * `50333697` start at sector 50333697
    * `[enter]` select the default end sector of 54525951
* `t` change partition type
    * `2` select partition #2
    * `82` change to swap
* `p` print _pending_ partition table
* `w` write partition table to disk and exit

Terminal session:

    root@ubuntu:~# fdisk /dev/sda

    WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
            switch off the mode (command 'c') and change display units to
            sectors (command 'u').

    Command (m for help): u
    Changing display/entry units to sectors

    Command (m for help): d
    Selected partition 1

    Command (m for help): n
    Command action
      e   extended
      p   primary partition (1-4)
    p
    Partition number (1-4): 1
    First sector (63-54525951, default 63): 2048
    Last sector, +sectors or +size{K,M,G} (2048-54525951, default 54525951): +24G

    Command (m for help): n
    Command action
      e   extended
      p   primary partition (1-4)
    p
    Partition number (1-4): 2
    First sector (50333697-54525951, default 50333697): 50333697
    Last sector, +sectors or +size{K,M,G} (50333697-54525951, default 54525951):
    Using default value 54525951

    Command (m for help): t
    Partition number (1-4): 2
    Hex code (type L to list codes): 82
    Changed system type of partition 2 to 82 (Linux swap / Solaris)

    Command (m for help): p

    Disk /dev/sda: 27.9 GB, 27917287424 bytes
    255 heads, 63 sectors/track, 3394 cylinders, total 54525952 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000524ec

      Device Boot      Start         End      Blocks   Id  System
    /dev/sda1            2048    50333696    25165824+  83  Linux
    /dev/sda2        50333697    54525951     2096127+  82  Linux swap / Solaris

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.

    WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
    The kernel still uses the old table. The new table will be used at
    the next reboot or after you run partprobe(8) or kpartx(8)
    Syncing disks.
    root@ubuntu:~# partprobe
    Warning: WARNING: partition(s) 1 on /dev/sda could not be modified, probably because it/they is/are in use.  As a result, the old partition(s) will remain in use until after reboot. You should reboot now before making further changes.

Since `fdisk` wrote the partition table but the kernel still uses the old one, so we must reboot:

    root@ubuntu:~# reboot

After the reboot, we can format the device for the second primary partition (`/dev/sda2`) as swap space:

    root@ubuntu:~# mkswap /dev/sda2
    Setting up swapspace version 1, size = 2096120 KiB
    no label, UUID=0ce65050-cc02-482b-9416-ef1bae404645

`mkswap` helpfully prints out the file system UUID, which we can then use to modify `/etc/fstab` to set the correct swap device:

    root@ubuntu:~# vi /etc/fstab # replace the above UUID with the new swap partition UUID

After modification, `/etc/fstab` will have one line for swap that looks like this:

    UUID=0ce65050-cc02-482b-9416-ef1bae404645 none            swap    sw              0       0

Now that the correct UUID is in place for swap, we turn it back on and display the swap configuration:

    root@ubuntu:~# swapon -a
    root@ubuntu:~# swapon -s
    Filename                                Type            Size    Used    Priority
    /dev/sda2                               partition       2096116 0       -1

Now all that is left to do is expand the `/` filesystem on `/dev/sda1` with `resize2fs` and print out the larger size and new disk layout:

    root@ubuntu:~# resize2fs /dev/sda1
    ...
    ...
    root@ubuntu:~# parted -s /dev/sda 'u GiB p'
    Model: VMware, VMware Virtual S (scsi)
    Disk /dev/sda: 26.0GiB
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos

    Number  Start    End      Size     Type     File system  Flags
    1      0.00GiB  24.0GiB  24.0GiB  primary  ext4
    2      24.0GiB  26.0GiB  2.00GiB  primary

Voila! Except that it requires a reboot!

In Part 2 I'll examine solutions to the reboot issue by discarding the use of partitions altogether when using virtual hardware.
