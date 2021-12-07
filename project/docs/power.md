
## Power-saving methods
On modern hardware Linux-based desktop environments favour a combination of the following methods for saving power:

1. monitor dimming
2. monitor suspension
3. session / system suspension to RAM (so-called _suspension_)
4. session / system suspension to disk (so-called _hibernation_)

_Monitor dimming_ (1), _Monitor suspension_ (2) and _Session suspension_ (3) should be easy to find: open your desktop environment's settings and search for _power management_. By contrast the last of these methods may require a few helpers.

## Suspension to disk (hibernation)

An openSUSE operating system is ready for hibernation only when:

- _Swap exists_: A swap space on the disk must exist (see below for details)
- _Swap referenced_: The swap space must be referenced in `/etc/fstab`, the configuration file read on every boot and used to mount and integrate the different spaces listed on the file into a proper file structure
- _Resume referenced_: The partition to which the user wants the hibernation to resume to must be made known to the kernel, usually as the value of a kernel parameter

Swap space can be created while partitioning the disk during installation or after installation.

__During installation__

This is adapted from [the installation documentation](yast_installer.md#about-partition-schemes) and repeated here for convenience.

<u>Step by step: Expert partitioning</u>

1. From the _Disk_ view, select _Expert Partitioner_
2. Click the __Add Partition__ button (bottom left-hand side)
3. Add a `swap` partition with a size equal to your current RAM + 1 GB.

__After installing__

From _YaST2_:

1. Click on, or search and then open, the _Partitioner_ module.
2. Select __Disks__ from the list on your left-hand side, and take note of the identifier of the partition you will want to resume to. Note both the UUID and the `/dev/sda<number>` identifiers.
3. Click on the __Create partition__ button. 
3. Add a `swap` partition with a size equal to your current RAM + 1 GB, making sure to format it as `swap`
4. Confirm to apply the changes.

Whichever method was used, by now YaST2 should have edited your `/etc/fstab` to reference the freshly created swap partition, which means that the conditions (Swap exists) and (Swap referenced) above should be met. You can double check that this is the case with

```
$ blkid | grep "swap"
```

This command should return a line referencing various identifiers for your swap partition. You can compare the UUID visible from this context with the UUID listed in _YaST2_ for good measure (see step 2 above). If you have created the swap space during installation, don't forget to search for the UUID and the `/dev/sda<number>` identifiers referencing the partition you are resuming to.

Finally, to fulfill condition (Resume referenced), in _YaST2_:

1. Click on, or search and then open, the _Bootloader_ module.
2. Switch to the __Kernel Parameters__ tab.
3. In the first text-field, labelled _Optional Kernel Parameters_, simply add the following text:
```
resume=/dev/sda<number> # replace '<number>' by the number of the target partition you want to resume to

```
or
```
resume=UUID=<UUID> # replace '<UUID>' with the UUID of the partition you want to resume to.
```

!!! warning
    It's preferable to use the UUID over any other identifier for referencing partitions as UUID identifiers are not ambiguous. However, should you recreate a resume or swap partition referenced via UUID, the reference will break and you will need to register it again where appropriate (kernel parameters if resume partition, `/etc/fstab` if swap space.) 

You can finally lay back and admire your configuration. On my machine it looks like this:

```
# kernel parameter
`resume=UUID=19bd024f-76bd-4162-ac37-4d0d51e02c25`
```
and 
```
# /etc/fstab
UUID=e0bac415-cbde-4c9b-8178-7874ac9de70a none swap defaults 0 0
```
