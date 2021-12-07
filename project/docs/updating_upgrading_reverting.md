## Introduction

Tumbleweed is a rolling-release distribution. It gets updates on a continuous basis, usually several times a week. However, to bring some order and make it easier to manage producing and consuming updates, Tumbleweed fetches updates in batches. A single batch of updates is called a _snapshot_.

!!! info
    Tumbleweed snapshots ought not to be confused with _Btrfs snapshots_, a completely different kind of snapshots. The latter make up the different filesystem states that allow _Btrfs_ users, using tools like YaST2 and `snapper`, to control which state they want to actualize.

Tumbleweed snapshots are thus batches of updates which are tested in openQA -- openSUSE's very best testing pipeline. The tests are thorough and check not only that each upgraded package is healthy, but also that all updates in the same snapshot play nice with one another. A snapshot is thus as healthy as the individual updates it holds, and as the compatibility between one another.

It should be emphasized that by default, Tumbleweed updates across all available updates, whether or not they belong to the same snapshots. From the user's point of view, snapshots succeed to one another seamlessly and are not supposed to disturb the user's workflow.

## Updating/upgrading

Unlike other Linux distributions, such as Leap, on Tumbleweed there is no distinction between updating (on other distributions: applying updates to packages) and upgrading (on other distributions: bringing the entire operating system to its latest possible state). You update/upgrade your system with:

```
sudo zypper dup
```

What this command does is to check whether all repositories registered against the system provide packages with available updates, collect and return the results. Upon confirmation from the user (`y` in a terminal window) it will download and then install them.

!!! info
    On Tumbleweed you will never have to use **zypper-up**, which is how you would update packages on Leap. The reason is that since Tumbleweed is a rolling-release distribution, updating it means not just updating an arbitrary collection of packages; it also means using the updated packages as _the new snapshot_ which Tumbleweed is to realize. In addition to this **zypper-dup** enjoys some extra power over **zypper-up**, being able to not only edit packages, but to remove them when they are no longer required.

It may occur that some repositories, when registered against the system, have not been configured to automatically look for new available updates during a `zypper-dup`. For this reason it might be useful to accompany this command with another one forcing all repositories to check for new updates, resulting in the pair:

`sudo zypper ref && sudo zypper dup`

That said, you can ensure that all repositories have been properly configured by listing them and, where applicable, by enabling their autorefresh feature. From the list of repositories obtained from:

`zypper lr`

identify repositories whose autorefresh value you want to tweak. For example:

```
#  | Alias                            | Name                | Enab-> | GPG Check        | Refresh
---+----------------------------------+---------------------+--------+-------------+-------------
 2 | brave-browser-beta               | brave-browser-beta  | Yes    | (r ) Yes         | No
```

shows that our `brave-browser-beta` repository has its refresh feature disabled. We can now set it to `enabled` with:

`sudo zypper mr -f 2`

which is a compact instance of 

`sudo zypper modifyrepo --refresh <repo identifier>` 

## Rolling back

On Tumbleweed _rolling back_ means changing the on-disk state of your system to a previous iteration. This is documented in details [there](snapper.md#rolling-back). In a nutshell:

1. Reboot your system. From the GRUB2 screen (bootloader), select __Start Bootloader from a read-only snapshot__.
2. Select the snapshot you want to boot to.
3. Log in normally and do `sudo snapper rollback`.

## Reverting to a previous kernel image

As it happens, the Linux kernel sometimes undergoes regressions. For the end user this often translates into issues where the system has trouble communicating with the hardware. The user could rollback to a previous _Btrfs_ snapshot (see previous section), but this would move the entire system -- not just the kernel -- to a previous state. However there are situations where only the kernel is the culprit. When that is the case you are often better off _reverting_ to an older kernel instead of clinging on an old Tumbleweed snapshot through a snapper rollback.

Here we will look at two likely scenarios:

1. You want to revert to a recent kernel version shortly after noticing issues with the latest kernel.
2. You want to revert to a not-so-recent kernel version because you weren't able to identify issues before it was removed from the official Tumbleweed repository.

For either scenario to work, you will need to configure your system to accept using an older kernel and to expose GRUB2 entries enabling you to use it persistently. So if you find yourself in the (1) or in the (2) scenario, please follow along:

Open zypper configuration file:

```
$   sudo nano /etc/zypp/zypp.conf
```

Make sure that the line `multiversion = provides:multiversion(kernel)` is not commented out (make sure there is no `#` prepended to `multiversion = provides:multiversion(kernel)`). If the line is commented out, use the arrow keys to bring the cursor near the `#` and delete it.

Then look at the line reading `multiversion.kernels = <version>`. It encodes the policy the system uses for keeping kernels. By default, what fills the `<...>` is `latest,latest-1,running`. This means that zypper should not remove:

- the kernel available from the latest Tumbleweed snapshot installed on the system; nor
- the one before; nor
- the current, running kernel

Once you have identified the exact kernel version you want to keep using for some time, simply add it as:

```
multiversion.kernels = latest,latest-1,5.13.12-2,running    # new: 5.13.12-2
```

!!! info
    Developers recommend specifying kernel versions in the format shown here, the _version_ proper i.e. `5.13.12-2`, instead of the longer `kernel-default-5.13.12-2` format, as the former is the standard format used by the `libzypp` library (the backend to zypper utilities).

Now the system should not remove:

- the kernel available from the latest Tumbleweed snapshot installed on the system; nor
- the one before; nor
- the kernel image named `5.13.12-2` (new)
- the current, running kernel

If you have followed through, your system now will _accept_ to use an old kernel __name__. Now you need to install and expose to the bootloader the __actual__ kernel image bearing the name.

### Reverting to an old but recent kernel still in the official repositories

If the kernel you have just allowed using the instructions from the previous section exists in the official repository, you simply have to do:

```
$   sudo zypper in <name of the kernel package>
```

where `<name of the kernel package>` could well be `5.13.12-2-default`.

Finally, reboot your system and pick the newly installed kernel from the corresponding GRUB2 entry:

__GRUB2__:

1. Select `Advanced options for openSUSE Tumbleweed`
2. Select the entry corresponding to the newly installed kernel.

### Reverting to an old kernel only present in Tumbleweed history

Unfortunately the official repositories do not store a wide gamut of kernels at any time. This means that if you notice that you need to revert to an old kernel after the kernel is gone from the official repositories, you need to manually download the kernel using [Tumbleweed's official "history" repositories](https://download.opensuse.org/history/).

Just like other packages, kernel packages are organized under Tumbleweed snapshots and CPU architecture. Thus to pick the right kernel package, you need to know:

1. the version of the kernel you need;
2. the Tumbleweed snapshot that shipped (1);
3. your CPU architecture.

Note that there is exactly one `kernel-default` per snapshot per CPU architecture.

These informations will enable you to find a directory on a path of type: `https://download.opensuse.org/history/<snapshot-you-need>/tumbleweed/repo/oss/<your-cpu-architecture>/`, and in it a `kernel-default-<version>.<your-cpu-architecture>.rpm` package. Download it and do:

```
$   sudo zypper in -f --oldpackage <path/to/kernel package.rpm>
```

`zypper` will take care of writing a new GRUB entry for the newly installed old kernel, but it is likely to place the new entry _below_ the entries for the more recent kernel entry or entries you are staying away from. If you want to set the new entry as default, follow the path below and select the entry listed at the end:

* _YaST_ > _Bootloader_ > _Bootloader Options_ > _Default Boot Section_ > __`<new kernel entry>`__ (select it)

You can now reboot and select the new entry and press `Enter` to start using the newly installed kernel.
