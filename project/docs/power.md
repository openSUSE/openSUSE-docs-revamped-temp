
## Power-saving methods
On modern hardware, Linux desktop environments generally use some combination of the following methods for saving power, often in this order:

1. Dimming the monitor(s)
2. Suspending the monitor(s)
3. Suspending the system into RAM (also called _suspend-to-RAM_, _sleep_ or _suspend mode_)
4. Suspending the system to disk (also called _suspend-to-disk_, _hibernation_ or _deep sleep_)

_Monitor dimming_, _monitor suspension_, and _suspend-to-RAM_ are usually enabled by default.
To check whether they are enabled, open your desktop environment's settings and search for _power management_.

In contrast, _hibernation_ may need to be set up manually.


## Suspend-to-disk (hibernation)

When it is running, your computer keeps all running programs and current state in memory (RAM).
Suspend-to-RAM is a way to save energy by powering off most components of a computer, except its memory, meaning state is kept.

This is a quick way to enable a low-power mode but has an important drawback:
It still requires a some power.
If you put your laptop to sleep for long that its battery runs out of power, the laptop will turn off.
This means you will lose any unsaved work and can also result in disk corruption.

A secondary issue is that some events can wake the machine from sleep, and there is a risk of it overheating in a confined space.
This includes connection and disconnection of USB devices or accidentally pressing external buttons or switches.

Suspend-to-disk (hibernation) is different:
The contents of the machine's memory are saved to disk, then the machine turns off completely.
In this state it uses no power and will resume back to where you were an arbitrary length of time later.

When Linux hibernates, it saves the contents of the memory to the swap partition.
On the next boot, the kernel checks the swap partition and if it finds a suspend image there, it reads it back into memory and resumes operation.

This has important consequences:

*   Your machine must have a _dedicated_ swap _partition_.

    *   If you use multiple Linux distributions side-by-side that share a swap partition, hibernation will have unpredictable results and will not work reliably.

    *   If you dual-boot with Windows, there can also be problems. Two potential issues are:

        *   If you use Windows' partition(s) under both OSes and hibernate Linux, boot Windows, and then resume Linux, this will result in disk corruption.
            This happens because the contents of the partitions will have changed while Linux was in hibernation.

        *   For related reasons, we recommend that you disable hibernation and "Fast Boot" in Windows.
            "Fast Boot" can cause issues if you mount any Windows partitions via `/etc/fstab`.
            To disable both settings, open command prompt in Administrator Mode and enter:
            ```
            powercfg -h off
            ```
            Then close the admin prompt and reboot.

*   Hibernation should but might not work if you use a swap _file_ in your root partition instead of a separate swap partition.

*   Hibernation will not work if you use the `ZRAM` tool to swap into compressed RAM disks.


### Requirements for hibernation

If you do have a dedicated partition, the following requirements must be met:

*   _Swap exists_: A swap partition on the disk must exist (see below for details)

*   _Swap is big enough_: the swap partition must be at least as big as the total amount of RAM fitted to the computer, and preferably slightly more.

    *   There must be enough space in the partition to hold the entire contents of your computer's memory.
        If your computer is short on RAM and so Linux normally uses swap space when running, then you need additional space for the memory image in addition to how much which is normally used.
        Historically, when RAM was expensive and systems typically used swap heavily, users often allocated at least twice as much space for swap as there was RAM.
        On modern computers with large amounts of RAM this usually is a waste of disk space.

    *   We recommend the size of RAM + 1 GB.

*   _Swap is referenced_: The swap space must be specified in `/etc/fstab`. This file specifies the various partitions to be mounted into the Linux file system and is read every time Linux boots.

*   _Resume is referenced_: The partition to which system will hibernate must be made known to the kernel, usually as the value of a kernel parameter


### Creating a swap partition

A swap partition can be created while partitioning your disks during installation, or in a separate step after installation.


#### During installation

This is adapted from [the installation documentation](yast_installer.md#about-partition-schemes) and repeated here for convenience.

<u>Step by step: Expert partitioning</u>

1.  From the _Disk_ view, select _Expert Partitioner_
2.  Click _Add Partition_ (bottom left-hand side)
3.  Add a `swap` partition with a size equal to your current RAM + 1 GB.


#### After installation

From _YaST_:

You need a dedicated disk partition for swap. It does not matter where on the disk it is, so after your existing partitions is often convenient.

If you have an existing partition you plan to use, you can skip the following two sections on partition creation.


##### If the partition does not exist yet

*   If your disk is partitioned with the GPT scheme, you can add another partition at the end.

*   If it is partitioned with MBR, the old DOS partitioning scheme applies:

    *   You can only have a maximum of four primary partitions, so you may want to make the new swap partition a logical partition inside the extended partition.

    *  If an extended partition does not already exist, create one.

    *  If you already have four primary partitions, you must remove one, or back up its contents to an external drive, remove it, create an extended partition and then copy its contents back into a new logical partition inside the extended partition.

    *  Logical partitions are numbered starting at `/dev/sda5`, even if you only have 1 or 2 primary partitions.


###### Creating a new swap partition

1.  Boot from a Linux Live USB, such as the [openSUSE Live CD](https://download.opensuse.org/distribution/leap/15.3/live/)

2.  Run the _GParted_ dynamic-partitioning tool.

3.  Choose a partition from which you can spare some capacity.

4.  Shrink your desired partition by the amount of space you wish to give to swap.

5.  Create a new partition in the now-free space. Pick a type of `Linux swap`.

6.  Reboot the computer and remove the bootable USB medium or optical disk.


##### If the partition already exists

1.  Open the _Partitioner_ module.

2.  Select _Disks_ from the list on your left-hand side, and take note of the identifier of the partition you will want to resume to.
    Note both the UUID and the `/dev/sda<number>` identifiers.

3.  Click _Create partition_.

4.  Add a `swap` partition with a size equal to your current RAM + 1 GB.
    Make sure it is formatted as `swap`.

5.  Confirm to apply the changes.


### Testing your new swap partition

Whichever method was used, by now _YaST_ should have edited your `/etc/fstab` file to reference the freshly created swap partition, which means that conditions 1 and 3 (swap exists and is referenced) above should be met.

You can check that this is the case with the command:

```
$ blkid | grep "swap"
```

This command should return a line referencing various identifiers for your swap partition.
You can compare the UUID visible from this context with the UUID listed in _YaST_ for good measure (see step 2 above).
If you created the swap partition during installation, search for the UUID and the `/dev/sda<number>` identifiers referencing the partition your computer will resume to.

If you have rebooted your computer after creating the partition, you can check to see if your new swap partition is active with the following command:

```
sudo swapon -s
```

If you have not yet rebooted and you wish to activate and start swapping onto the partition immediately, use the following command with the device name of your new partition:

```
sudo swapon /dev/sda<number>
```

Finally, make sure that the resume partition is referenced via _YaST_:

1.  Open the _Bootloader_ module.

2.  Switch to the _Kernel Parameters_ tab.

3.  In the text field _Optional Kernel Parameters_, add the `resume` parameter and a reference to UUID of the partition you want to resume from:
    ```
    resume=UUID=<UUID> # replace '<UUID>' with the UUID
    ```
    Alternatively, you can use the partition number _(not recommended)_:
    ```
    resume=/dev/sda<number> # replace '<number>' with the partition number
    ```

!!! important
    We recommend using UUIDs (_Universal Unique Identifier_) instead of other identifiers for referencing partitions.
    UUIDs are unambiguous and do not change when other partitions are added or removed.

    However, UUIDs will change if you reformat or delete and recreate a swap partition.
    In such cases, update the UUID where appropriate.
    For resume partitions, update the `resume` kernel parameter:
    ```
    resume=UUID=19bd024f-76bd-4162-ac37-4d0d51e02c25
    ```
    For swap space, update `/etc/fstab`:
    ```
    UUID=e0bac415-cbde-4c9b-8178-7874ac9de70a none swap defaults 0 0
    ```

When you are happy that the partition is in place and works, try the hibernate command of your desktop.
This is often located next to the shut down, reboot and sleep commands.


## More information

You can read about testing and using hibernation functionality in the openSUSE Wiki:
[SDB:Suspend to disk](https://en.opensuse.org/SDB:Suspend_to_disk).
