
## Power-saving methods
On modern hardware, Linux desktop environments generally use some combination of the following methods for saving power, often in this order:

1. dimming the monitor(s)
2. putting the monitor(s) to sleep
3. suspending the system into RAM (often called "sleep" or "suspend mode")
4. suspending the system to disk (usually called _hibernation_)

The first three of these — _monitor dimming_ (1), _monitor suspension_ (2), and _session suspension_ (3) should be easy to find: open your desktop environment's settings and search for _power management_. In contrast, the last of these methods may require some extra configuration.


## Suspend to disk (hibernation)

Suspend to RAM, or sleep, is easy and fast, but it has an important drawback: it still requires a small amount of power. If you put your laptop to sleep and put it away, then when its battery runs out of power, the laptop will turn off. You will not only lose any unsaved work, but this counts as an unclean shutdown and can lead to disk corruption. A secondary issue is that some events, including connection and disconnection of USB devices, the accidental pressing of any external buttons or switches, can wake the machine from sleep and there is a risk of it overheating in a confined space.

Hibernation (called "deep sleep" in Apple macOS) is different: the contents of the machine's memory are saved to disk and then the machine turns off completely. In this state it uses no power and will resume back to where you were an arbitrary length of time later.

When Linux hibernates, it stores the contents of memory in your swap partition. When you next boot the machine, when the kernel starts up, it looks in the swap partition and if it finds a suspend image there, it reads it back into memory and resumes operation.

 This has some important consequences:
 * You must have a _dedicated_ swap _partition_.
     * Hibernation will have unpredictable results and will not work reliably if you dual-boot two or more different distributions and share a swap partition between them.
     * If you dual-boot with Windows, you may also have problems. Two potential issues are:
        * You hibernate Linux, boot into Windows, then subsequently try to resume Linux. If your Windows partition(s) were mounted, their contents will have changed while Linux was in hibernation, and disk corruption will probably result.
        * For related reasons,  we recommend that you disable hibernation in Windows. (This also disables Windows' "Fast Boot" feature, which is desirable as it can cause problems if you mount any Windows problems from `/etc/fstab`.) Open an Admin Mode command prompt and type:
        ```
        powercfg -h off
        ```
        Then close the admin prompt and reboot.
 * Hibernation will not work if you use a swap _file_ in your root partition instead of a separate swap partition.
 * Hibernation will not work if you use the `ZRAM` tool to swap into compressed RAMdisks.

### Requirements for hibernation

If you do have a dedicated partition, the following requirements must be met:
1. _Swap exists_: A swap partition on the disk must exist (see below for details)
2. _Swap is big enough_: the swap partition must be at least as big as the total amount of RAM fitted to the computer, and preferably slightly more.
    * There must be enough space in the partition to hold the entire contents of your computer's memory. If your computer is short on RAM and so Linux normally uses swap space when running, then you need additional space for the memory image in addition to how much which is normally used. Historically, when RAM was expensive and systems typically used swap heavily, it was traditional to allocate at least twice as much space for swap as there was RAM, but on modern computers with plentiful RAM this would usually be a waste of disk space.
    * We recommend the size of RAM + 1 GB.
3. _Swap is referenced_: The swap space must be specified in `/etc/fstab`. This file specifies the various partitions to be mounted into the Linux file system and is read every time Linux boots.
4. _Resume referenced_: The partition to which system will hibernate must be made known to the kernel, usually as the value of a kernel parameter

### Creating a swap partition

A swap partition can be created while partitioning your disks during installation, or in a separate step after installation.

#### During installation

This is adapted from [the installation documentation](/yast_installer#about-partition-schemes) and repeated here for convenience.

<u>Step by step: Expert partitioning</u>

1. From the _Disk_ view, select _Expert Partitioner_
2. Click the "Add Partition" button (bottom left-hand side)
3. Add a `swap` partition with a size equal to your current RAM + 1 GB.

#### After installation

From _Yast2_:

You need a dedicated disk partition for swap. It does not matter where on the disk it is, so after your existing partitions is often convenient.

If you have an existing partition you plan to use, you can skip the following two sections on partition creation.

##### If the partition does not exist yet

* If your disk is partitioned with the GPT scheme: just add another partition on the end.
* If it is partitioned with MBR, the old DOS partitioning scheme: you can only have a maximum of four primary partitions, so you may want to make the new swap partition a logical partition inside the extended partition. If an extended partition does not already exist, create one. If you already have four primary partitions, you must remove one, or back up its contents to an external drive, remove it, create an extended partition and then copy its contents back into a new logical partition inside the extended partition. Logical partitions are numbered starting at `/dev/sda5`, even if you only have 1 or 2 primary partitions.

###### Creating a new swap partition

1. Boot from a Linux Live USB, such as the [openSUSE Live CD](https://download.opensuse.org/distribution/leap/15.3/live/)
2. Run the _GParted_ dynamic-partitioning tool.
3. Choose a partition from which you can spare some capacity.
4. Shrink your desired partition by the amount of space you wish to give to swap.
5. Create a new partition in the now-free space. Pick a type of `Linux swap`.
6. Reboot the computer and remove the bootable USB medium or optical disk.

##### If the partition already exists

1. Click on, or search and then open, the _Partitioner_ module.
2. Select "Disks" from the list on your left-hand side, and take note of the identifier of the partition you will want to resume to. Note both the UUID and the `/dev/sda<number>` identifiers.
3. Click on the "Create partition" button.
3. Add a `swap` partition with a size equal to your current RAM + 1 GB, making sure to format it as `swap`
4. Confirm to apply the changes.

### Testing your new swap partition

Whichever method was used, by now Yast2 should have edited your `/etc/fstab` file to reference the freshly created swap partition, which means that conditions № 1 and 3 (Swap exists and is referenced) above should be met.

You can double-check that this is the case with the command:

```
$ blkid | grep "swap"
```

This command should return a line referencing various identifiers for your swap partition. You can compare the UUID visible from this context with the UUID listed in _Yast2_ for good measure (see Step 2 above). If you created the swap partition during installation, remember to search for the UUID and the `/dev/sda<number>` identifiers referencing the partition your computer will resume to.

If you have rebooted your computer after creating the partition, you can check to see if your new swap partition is active with the following command:
```
sudo swapon -s
```
If you have not yet rebooted and you wish to activate and start swapping onto the partition immediately, use the following command with the device name of your new partition:

```
sudo swapon /dev/sda<number>
```

Finally, to fulfil condition № 4 (Resume is referenced), in _Yast2_:

1. Click on, or search and then open, the _Bootloader_ module.
2. Switch to the "Kernel Parameters" tab.
3. In the first text-field, labelled _Optional Kernel Parameters_, simply add the following text:
```
resume=/dev/sda<number> # replace '<number>' by the number of the target partition you want to resume to
```
or
```
resume=UUID=<UUID> # replace '<UUID>' with the UUID of the partition you want to resume to.
```

> :point_right: **Tip: UUIDs are preferable to device names**
>
> It is preferable to use the UUID over any other identifier for referencing partitions, because UUID identifiers are not ambiguous and do not change if other partitions are added or removed. UUIDs will change if you reformat or delete and recreate a swap partition. You will need to re0-enter  it again where appropriate (kernel parameters if resume partition, `/etc/fstab` if swap space.)

Check your settings. On my machine, it looks like this:

```
# kernel parameter
`resume=UUID=19bd024f-76bd-4162-ac37-4d0d51e02c25`
```
and
```
# /etc/fstab
UUID=e0bac415-cbde-4c9b-8178-7874ac9de70a none swap defaults 0 0
```
``

Once you are happy that the partition is in place and works, try the Hibernate command in your desktop. This is often located next to the Shut down, Reboot and Sleep commands.

## More information

You can read about testing and using hibernation functionality in the openSUSE Wiki:

[SDB:Suspend to disk](https://en.opensuse.org/SDB:Suspend_to_disk)
