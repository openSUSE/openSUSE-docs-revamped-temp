## Zypper

Zypper is a command line package manager for installing, updating and removing packages. It also manages repositories, can perform various queries, and is a command-line interface to ZYpp system management library (libzypp).

???+ info "Show Definitions"
    `repository`
    :   Local or remote directory containing packages and various package information (package metadata).

    `package`
    :   Software package (RPM package, source package). Patches, patterns, and products are also commonly referred to as different types of packages.

    `patch`
    :   Represents a package or a group of packages needed to install an update (a bug fix mostly).

    `pattern`
    :   Represents a group of packages. For example an Http Server pattern would have the dependencies defined so that all packages needed to run and manage an http server would have been triggered for installation when upon installing the pattern.

    `product`
    :   Represents the whole product (e.g. 'openSUSE Leap 15.3').

### What is a Package Manager?
Software is incredibly interconnected. Programming a user interface, a developer's code will specify the location on the screen
of widgets, such as buttons or text, and define how these things rearrange when a window is resized. But most code depends on
some other code that serves as a 'backend' (often called a _library_). For example, an application will rarely describe how to
render a button, instead it will ask a library to do that for it. This allows for a number of benefits. Some people specialise
on writing button drawing routines while others specialise on using those routines to
create feature-full applications.

Apps written for Linux use this principle a lot and it opens up a lot of opportunity. If 100 apps all use the same
library, we could install that library once and all 100 apps could use that single copy when running. This has
the following benefits:

  1. We save space! Instead of every graphical application containing the code necessary for drawing a button,
  we have that code installed once!
  2. If a library used by many is found to contain insecure, buggy or slow code, you can fix and update the library and every 
  app that uses it gains that improvement. We don't have to rely on developers tracking the libraries that they use
  and rebuilding their apps every time one changes.
  3. If you improve upon a library in some way, you can exchange the version on your computer with
  your modified version. Your improvement is then available to all applications that use that library and can adopt your changes. 
  This is a core principle in [free software](https://en.wikipedia.org/wiki/Free_software).

However, there is a drawback. When installing software, we have to ensure that the libraries that it requires (known as dependencies)
are also installed. When removing software, we cannot remove a dependency without checking to make sure that
nothing else depends upon it -- or it will break the 'dependency-tree' of the operating system. To visualize the issue, consider the
following illustration:
```
    lib5      lib6
   /    \    /    \
lib1     lib3      lib4
  \      /  \      /
  softwareA  softwareB
```
Let's assume for simplicity that no other software requires `lib4`. Under this assumption, if the user remove softwareB, they will want
to remove `lib4`, but neither `lib3` nor `lib6`, because the former is directly required by `softwareA`, while the latter is
indirectly required by `softwareA` via `lib3`.

In addition to this issue imagine the user updates `softwareB` so that it ends up depending on a new version of library `lib4`,
say `lib4*`. This means that we need to recast the above structure as:
```
    lib5      lib6
   /    \    /    \
lib1     lib3      lib4       lib4*
  \      /  \                 /
  softwareA  softwareB(updated)
```
This means that the system must be able to 'see through' the actual _versions_ of each package, so as to understand the path of
their dependencies and warn about required alterations.

These problems are solved with a package manager. Developers can build applications, libraries and many other things into packages that
contain what you want to install and a set of references to any other packages that it depends upon. The _package manager_ is an essential
piece of software that downloads, installs, uninstalls and updates packages for you. If you tell the package manager to install Firefox,
it will download a package containing the Firefox executable, as well as a number of packages that contain the dependencies
of Firefox. It also holds a database of all installed packages so it knows when it can uninstall certain dependencies without
breaking software on your computer. _Zypper_ is the name of the package manager used by openSUSE and SUSE Enterprise Linux.


### Usage
Zypper is a command line tool, meaning that it is used from within the terminal. Installing, updating and removing packages are operations that affect all users on the system and so many of these commands will not work unless they are run as a privileged user. Although special permissions can be granted to individual user accounts, it is simpler to run commands as the root user (the user with the most privilege). Prefixing a command with `sudo` (e.g. `sudo zypper install firefox`) will ask for the password of the `root` user account and then runs the command as `root`.

!!! info
    The components enclosed in brackets are not required, thus the simplest way to execute zypper is to type its name followed by a command. See zypper help for a list of general options and all commands. To get help for a specific command, type zypper help command

```
# zypper [--GLOBAL-OPTIONS] <COMMAND> [--COMMAND-OPTIONS] [ARGUMENTS]
```
```
# zypper <SUBCOMMAND> [--COMMAND-OPTIONS] [ARGUMENTS]`
```

!!! note on User Prompts
    Whenever Zypper needs input, it lists possible answers in brackets next to the prompt text. To choose the default answer, press &lt;__Enter__&gt; (the default answer is printed in capitals, with the exception of non-ascii characters). Some prompts also have help available, in which case there is a question mark '?' listed as a possible answer.

    To make Zypper use the default answers without user interaction, use the `--non-interactive` global option.

### Commands    
A single command is given to zypper (without a leading `--`) to instruct it on what kind of task
it should do. For example `zypper install` instructs zypper that we want to install something and
`zypper search` instructs zypper that we want to search for a package.
!!! tip
    The most used zypper commands have shorter aliases; for example, `zypper in` is the same
    as `zypper install`.

    ``zypper`` without any commands or arguments will print a list of commands and options.

This manual is a great starting point but it does not cover all possible ways to use zypper. If you want to see all of the options
that can be given to each subcommand in zypper, type:
```
zypper <COMMAND> -h
```
and it will show all of the different ways of configuring the command specified in an easy to read way.

#### The Basics

##### Installing packages
To install a package type:
```
# zypper install packagename
```
or
```
# zypper in packagename
```

This will download the package you specified along with all of
its dependencies and install them. If you want to install more than
one package, you simply list them all:
```
# zypper in package_1 package_2 ... package_n
```

##### Searching for packages
In order to install a package, you need to know its name. To search the repositories
for a package type:
```
# zypper search packagename
```
or
```
# zypper se packagename
```
This will return a list of packages that contain your search string along
with a summary, a type and whether they have been installed.
???+ Example "Example: Search for Firefox"
    ```
    # zypper se firefox
    Loading repository data...
    Reading installed packages...

    S  | Name                               | Summary                                      | Type
    ---+------------------------------------+----------------------------------------------+--------
    i+ | MozillaFirefox                     | Mozilla Firefox Web Browser                  | package
    i+ | MozillaFirefox-branding-openSUSE   | openSUSE branding of MozillaFirefox          | package
       | MozillaFirefox-branding-upstream   | Upstream branding for Firefox                | package
       | MozillaFirefox-buildsymbols        | Breakpad buildsymbols for Firefox            | package
       | MozillaFirefox-devel               | Devel package for Firefox                    | package
       | MozillaFirefox-translations-common | Common translations for Firefox              | package
       | MozillaFirefox-translations-other  | Extra translations for Firefox               | package
       | eid-mw-firefox                     | Firefox Extension for Belgium eID Middleware | package
       | firefox-esr-branding-openSUSE      | openSUSE branding of MozillaFirefox          | package
       | firefox-uget-integrator            | Integration of uGet with Firefox             | package
    ```
    Note: `i` or `i+` tell us that it is already installed. `i+` means installed by request of the user
    and `i` means it was installed as a dependency

##### Adding a New Repository
A repository is simply a source of packages. Zypper will already have a few
added (the default for either Tumbleweed or Leap) but there are many reasons
why you might want to add some more. You might need to install proprietary
libraries such as cuda or non-free media codecs. To add a new repository type:
```
zypper addrepo https://somerepo.com/something name
```
or
```
zypper ar https://somerepo.com/something [optional-name]
```
If successful, zypper will report the settings of that repo:

  1. URI of the repository
  2. If the repository is enabled
  3. If the repository will be GPG checked
  4. The priority

???+ info "Understanding The Output"
    `Enabled`/`Disabled`
    : If a repository is disabled, it means that zypper still knows where it is
    but will not use it. You can enable repositories globally or for single
    commands.

    `GPG check`
    : If a repository has GPG checks enabled, it means that your computer will
    save a key, which you will be asked if you trust it the first time you use a repository.
    After downloading a package, zypper will use this key to make sure that the
    download succeeded and the package is not corrupted. It also verifies that
    the package is authentic. If a hacker intercepts a package download in an attempt
    to load malicious software onto your computer, the evil package will fail the
    GPG check.

    `Priority`
    : If two repositories contain the same package and you ask zypper to install it,
    it will automatically use a repository with a higher priority (lower number).

    `Autorefresh`
    : If autorefresh is on, zypper will automatically download a fresh list
    of packages from the repository to ensure that it isn't going to give
    you out of date information or packages.

???+ Example "Example: Adding the cuda Leap repo"
    ```
    # sudo zypper ar https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/cuda-opensuse15.repo cuda
    [sudo] password for root:
    Adding repository 'cuda' ..................................................................................[done]
    Repository 'cuda' successfully added

    URI         : https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64
    Enabled     : Yes
    GPG Check   : Yes
    Autorefresh : No
    Priority    : 99 (default priority)

    Repository priorities are without effect. All enabled repositories share the same priority.
    ```

##### Removing Repositories
You might decide that you no longer need a repository. Maybe you broke
your nvidia graphics card and feel that having the cuda repository
just slows down zypper operations. You can remove a repository typing
```
# zypper removerepo reponame
```
or
```
# zypper rr reponame
```
???+ Example "Example: Removing the cuda repository"
    ```
    # sudo zypper rr cuda
    [sudo] password for root:
    Removing repository 'cuda' ...............................................[done]
    Repository 'cuda' has been removed.
    ```

##### Listing Repositories
To list all of the installed repositories type:
```
# zypper repos
```
```
# zypper lr
```
This will display a table containing information and names of all the
added repos

???+ Example "Example: Listing repositories"
    ```
    # zypper lr
    Repository priorities are without effect. All enabled repositories share the same priority.

    # | Alias             | Name                      | Enabled | GPG Check | Refresh
    --+-------------------+---------------------------+---------+-----------+--------
    1 | repo-debug        | Debug Repository          | No      | ----      | ----
    2 | repo-debug-update | Update Repository (Debug) | No      | ----      | ----
    3 | repo-oss          | Main Repository           | Yes     | (r ) Yes  | Yes
    4 | repo-source       | Source Repository         | No      | ----      | ----
    5 | repo-update       | Main Update Repository    | Yes     | (r ) Yes  | Yes
    ```
!!! info
    See the info box in [Adding a New Repository](#adding-a-new-repository) to
    understand the output

##### Removing Packages
Sometimes you will want to remove packages. To achieve this, type:
```
# zypper remove package
```
or
```
# zypper rm package
```
!!! tip
    If your aim is to save space, you may want to add `-u` or `--clean-deps`
    e.g. `sudo zypper rm -u firefox` to remove the unneeded dependencies
    as well.

Just like when installing, you can remove multiple packages at a time:
```
# zypper rm package_1 package_2 ... package_n
```

##### Refreshing your repositories
Your computer stores a list of packages that each repository provides and metadata to help it work out
what each package depends on. This is so that when you instruct zypper to do something, it can calculate
the steps it will take without having to make many slow web requests. For some repositories,
this offline list is automatically kept up to date (called autorefresh), but a lot of the time you will
want to update this offline list yourself. Before updating, after adding a repository and before
installing a package it is a good idea to update this list. You update it by typing:
```
# zypper refresh
```
or
```
# zypper ref
```

#### Updating Your Computer
There are three different types of update that you can do with zypper

  1. Standard Update
  2. Patch Updates
  3. Distro Upgrade

Standard updates simply check to see if newer versions of your packages exist in the repository.
If there is a newer version, it will download and install it.

Patches are groups of package updates. They are categorised so that you can see why they are being updated. For example,
often, when you ask zypper to list available patches, some will be labelled as security patches. This  means that they
fix a certain security problem that has been discovered in some packages on your system.

Distro upgrades do not only update the packages on your system, but also change the source that you get your packages from.
This is used mostly on Tumbleweed. The build server where Tumbleweed packages are built is a bit chaotic; packages
are updated at an extremely high speed. A consequence of this is that incompatibilities between new packages emerge all over the place. So how does
Tumbleweed stay so stable? In regular intervals, a stable snapshot of the Tumbleweed packages is found and frozen for
a short amount of time.

Tumbleweed users use the distro upgrade feature of zypper to get their updates and so they change their
distribution to the newest snapshot. This means that zypper changes repositories to point to the newer snapshot, and then updates all the packages
with respect to the new snapshot. This means that Tumbleweed users can enjoy incredibly up to date packages without having to deal with
too much instability. Distro-upgrades can be used by leap users to upgrade version (for example, upgrading Leap 15.1 to 15.2). See 
[here](https://en.opensuse.org/SDB:System_upgrade) for more details.

Don't be intimidated by the number of options available to you for updating. The golden rule is if you are on Tumbleweed, do distro-upgrades
and if you are on leap do normal upgrades. The exception to this rule is if you are in charge of a production system, such as a web-server
(or any system where the only things scarier than change are security issues)
then use patches to ensure that only the minimum amount of your system is touched by updates.

##### Listing Updates
To list all of packages that have updates waiting for them, type:
```
# zypper list-updates
```
or
```
# zypper lu
```
Zypper will print out a table that shows you the updatable packages, the repos they come from, the installed versions and
the available versions.

???+ Example "Example: Listing updates"
    ```
    # zypper lu
    Loading repository data...
    Reading installed packages...
    S | Repository             | Name             | Current Version      | Available Version    | Arch
    --+------------------------+------------------+----------------------+----------------------+--------
    v | Main Update Repository | grub2            | 2.04-lp152.7.18.2    | 2.04-lp152.7.22.7    | aarch64
    v | Main Update Repository | grub2-arm64-efi  | 2.04-lp152.7.18.2    | 2.04-lp152.7.22.7    | noarch
    v | Main Update Repository | hwdata           | 0.343-lp152.2.3.1    | 0.345-lp152.2.6.1    | noarch
    v | Main Update Repository | libefivar1       | 37-lp152.3.6.1       | 37-lp152.3.9.1       | aarch64
    v | Main Update Repository | libsolv-tools    | 0.7.16-lp152.2.13.1  | 0.7.17-lp152.2.16.1  | aarch64
    v | Main Update Repository | libz1            | 1.2.11-lp152.8.6.1   | 1.2.11-lp152.8.9.1   | aarch64
    v | Main Update Repository | libzypp          | 17.25.6-lp152.2.19.1 | 17.25.8-lp152.2.22.1 | aarch64
    v | Main Update Repository | python3-solv     | 0.7.16-lp152.2.13.1  | 0.7.17-lp152.2.16.1  | aarch64
    v | Main Update Repository | ruby-solv        | 0.7.16-lp152.2.13.1  | 0.7.17-lp152.2.16.1  | aarch64
    v | Main Update Repository | tcl              | 8.6.7-lp152.7.3.1    | 8.6.7-lp152.7.6.1    | aarch64
    v | Main Update Repository | tk               | 8.6.7-lp152.4.3.1    | 8.6.7-lp152.4.6.1    | aarch64
    v | Main Update Repository | yast2-security   | 4.2.19-lp152.2.12.1  | 4.2.23-lp152.2.15.1  | noarch
    v | Main Update Repository | yast2-storage-ng | 4.2.114-lp152.2.9.1  | 4.2.115-lp152.2.12.1 | aarch64
    v | Main Update Repository | zypper           | 1.14.42-lp152.2.15.1 | 1.14.43-lp152.2.18.1 | aarch64
    v | Main Update Repository | zypper-aptitude  | 1.14.42-lp152.2.15.1 | 1.14.43-lp152.2.18.1 | noarch
    ```

##### Updating
In order to install all necessary updates, you can simply type:
```
# zypper update
```
or
```
# zypper up
```
You also have the option to update just a select few packages. And this is done, just like with installing
and removing, by listing the packages you want to update after the command:
```
# zypper up package_1 package_2 ... package_n
```

##### Checking for patches
To check for available patches, type:
```
# zypper patch-check
```
or
```
# zypper pchk
```
This will print out a table that shows the number of patches available for your system and how important they are

???+ Example "Example: Checking for patches"
    ```
    # sudo zypper pchk
    Loading repository data...
    Reading installed packages...

    Found 8 applicable patches:
    Category    | Updatestack | Patches
    ------------+-------------+--------
    security    | -           | 1
    recommended | 1           | 5
    optional    | -           | 1

    1 patch optional                                 (use '--with-optional' to include optional patches)
    7 patches needed (1 security patch)
    ```

##### Listing patches
If you looked at the example in the section above, you may have noticed that very little information is
given about the patches. To see much better information about the patches themselves, type:
```
# zypper list-patches
```
or
```
# zypper lp
```
This will output at least one table showing you the name, id, priority and a little note about each set of patches.

???+ Example "Example: Listing patches"
    ```
    # zypper lp
    Loading repository data...
    Reading installed packages...

    Needed software management updates will be installed first:

    Repository             | Name              | Category    | Severity | Interactive | Status | Summary
    -----------------------+-------------------+-------------+----------+-------------+--------+------------------------------------------------
    Main Update Repository | openSUSE-2021-456 | recommended | moderate | restart     | needed | Recommended update for libsolv, libzypp, zypper

    The following updates are also available:

    Repository             | Name              | Category    | Severity  | Interactive | Status   | Summary
    -----------------------+-------------------+-------------+-----------+-------------+----------+----------------------------------------
    Main Update Repository | openSUSE-2021-361 | optional    | low       | ---         | optional | Optional update for tk and tcl
    Main Update Repository | openSUSE-2021-433 | recommended | moderate  | ---         | needed   | Recommended update for yast2-storage-ng
    Main Update Repository | openSUSE-2021-434 | recommended | moderate  | ---         | needed   | Recommended update for yast2-security
    Main Update Repository | openSUSE-2021-438 | recommended | moderate  | ---         | needed   | Recommended update for efivar
    Main Update Repository | openSUSE-2021-439 | recommended | moderate  | ---         | needed   | Recommended update for zlib
    Main Update Repository | openSUSE-2021-462 | security    | important | ---         | needed   | Security update for grub2
    Main Update Repository | openSUSE-2021-463 | recommended | low       | ---         | needed   | Recommended update for hwdata

    Found 8 applicable patches:
    1 patch optional                                                                          (use '--with-optional' to include optional patches)
    7 patches needed (1 security patch)
    ```

##### Installing a patch
To install all available patches type:
```
# zypper patch
```
If you want to specify a specific category of patches, type:
```
# zypper patch -g CATEGORY
```

???+ Example "Example: Applying only security patches"
    ```
    # sudo zypper patch -g security
    Loading repository data...
    Reading installed packages...
    Patch 'openSUSE-2021-456-1' is not in the specified category.
    Patch 'openSUSE-2021-463-1' is not in the specified category.
    Patch 'openSUSE-2021-456-1' is not in the specified category.
    Patch 'openSUSE-2021-439-1' is not in the specified category.
    Patch 'openSUSE-2021-438-1' is not in the specified category.
    Patch 'openSUSE-2021-434-1' is not in the specified category.
    Patch 'openSUSE-2021-433-1' is not in the specified category.
    Patch 'openSUSE-2021-361-1' is optional. Use 'zypper in patch:openSUSE-2021-361' to install it, or '--with-optional' to include all optional patches.
    Resolving package dependencies...

    The following 3 NEW packages are going to be installed:
      dmraid grub2-systemd-sleep-plugin os-prober

    The following NEW patch is going to be installed:
      openSUSE-2021-462

    The following 2 packages are going to be upgraded:
      grub2 grub2-arm64-efi

    2 packages to upgrade, 3 new.
    Overall download size: 3.7 MiB. Already cached: 0 B. After the operation, additional 706.9 KiB will be used.
    Continue? [y/n/v/...? shows all options] (y): y
    Retrieving package dmraid-1.0.0.rc16-lp152.5.3.aarch64                                                 (1/5), 133.8 KiB (528.0 KiB unpacked)
    Retrieving: dmraid-1.0.0.rc16-lp152.5.3.aarch64.rpm ..................................................................................[done]
    Retrieving package os-prober-1.76-lp152.3.2.aarch64                                                    (2/5),  51.6 KiB (188.4 KiB unpacked)
    Retrieving: os-prober-1.76-lp152.3.2.aarch64.rpm .....................................................................................[done]
    Retrieving package grub2-2.04-lp152.7.22.7.aarch64                                                     (3/5),   2.6 MiB ( 23.9 MiB unpacked)
    Retrieving: grub2-2.04-lp152.7.22.7.aarch64.rpm ..........................................................................[done (1.7 MiB/s)]
    Retrieving package grub2-systemd-sleep-plugin-2.04-lp152.7.22.7.noarch                                 (4/5),  64.6 KiB (  7.1 KiB unpacked)
    Retrieving: grub2-systemd-sleep-plugin-2.04-lp152.7.22.7.noarch.rpm ..................................................................[done]
    Retrieving package grub2-arm64-efi-2.04-lp152.7.22.7.noarch                                            (5/5), 885.3 KiB (  5.1 MiB unpacked)
    Retrieving: grub2-arm64-efi-2.04-lp152.7.22.7.noarch.rpm .................................................................[done (2.2 MiB/s)]

    Checking for file conflicts: .........................................................................................................[done]
    (1/5) Installing: dmraid-1.0.0.rc16-lp152.5.3.aarch64 ................................................................................[done]
    (2/5) Installing: os-prober-1.76-lp152.3.2.aarch64 ...................................................................................[done]
    (3/5) Installing: grub2-2.04-lp152.7.22.7.aarch64 ....................................................................................[done]
    (4/5) Installing: grub2-systemd-sleep-plugin-2.04-lp152.7.22.7.noarch ................................................................[done]
    (5/5) Installing: grub2-arm64-efi-2.04-lp152.7.22.7.noarch ...........................................................................[done]
    ```

##### Distro upgrading
To do a distro upgrade, type:
```
# zypper dist-upgrade
```
or
```
# zypper dup
```

#### More advanced commands
If you are digging into these commands, then I think we can safely assume that you know what you are doing.
All zypper commands have a plethora of options, which can be discovered in the man pages, or by adding
`-h` to the end a subcommand invocation. This section is more to introduce you to things zypper can do!
##### Rename specified repository
* `renamerepo` or `nr`
##### Modify specified repository
* `modifyrepo` or `mr`

##### Clean local caches
* `clean` or `cc`

##### Verify integrity of package dependencies
* `verify` or `ve`
##### Install source packages and their build dependencies
* `source-install` or `si`
##### Install newly added packages recommended by installed packages
* `install-new-recommends` or `inr`


##### Show full information for specified packages
* `info` or `if`

!!! example
    ``$ sudo zypper if [package]``
##### Show full information for specified patches
* `patch-info`
##### Show full information for specified patterns
* `pattern-info`
##### Show full information for specified products
* `product-info`
##### List all available patches
* `patches` or `pch`
##### List all available packages
* `packages` or `pa`
##### List all manually installed packages as a single history
For the entire list:

`sudo cut -d "|" -f 1-4 -s --output-delimiter " | " /var/log/zypp/history | grep -v " radd "`

For a specific package:

`sudo cut -d "|" -f 1-4 -s --output-delimiter " | " /var/log/zypp/history | grep -v " radd " | grep "<package name>"`

##### List all available patterns
* `patterns` or `pt`
##### List all available products
* `products` or `pd`

???+ example
    ```
    # zypper pd
    Loading repository data...
    Reading installed packages...
    S  | Repository                  | Internal Name         | Name                  | Version    | Arch   | Is Base
    ---+-----------------------------+-----------------------+-----------------------+------------+--------+--------
    i+ | @System                     | openSUSE              | openSUSE Tumbleweed   | 20201216-0 | x86_64 | Yes
       | openSUSE-Tumbleweed-Non-Oss | openSUSE-Addon-NonOss | openSUSE NonOSS Addon | 20201218-0 | x86_64 | No
       | openSUSE-Tumbleweed-Non-Oss | openSUSE-Addon-NonOss | openSUSE NonOSS Addon | 20201218-0 | i586   | No
    v  | openSUSE-Tumbleweed-Oss     | openSUSE              | openSUSE Tumbleweed   | 20201218-0 | x86_64 | No
    v  | openSUSE-Tumbleweed-Oss     | openSUSE              | openSUSE Tumbleweed   | 20201218-0 | i586   | No
       | openSUSE-Tumbleweed-Oss     | MicroOS               | openSUSE MicroOS      | 20201218-0 | x86_64 | No
       | openSUSE-Tumbleweed-Oss     | MicroOS               | openSUSE MicroOS      | 20201218-0 | i586   | No
    ```
##### List packages providing specified capability
* `what-provides` or `wp`

!!! warning
    The subcommand `what-provides` is deprecated. It is replaced by `search --provides --match-exact`.

##### Add a package lock
* `locks` or `ll`
##### Remove a package lock
* `addlock` or `al`
##### List current package locks
* `removelock` or `rl`
##### Remove useless locks
* `cleanlocks` or `cl`
##### List requested locales (languages codes)
* `locales` or `lloc`
##### Add locale(s) to requested locales
* `addlocale` or `aloc`
##### Remove locale(s) from requested locales
* `removelocale` or `rloc`
##### Get command-specific help
* `help`

!!! example
    ``$ zypper help in``
##### Compare two version strings
* `versioncmp` or `vcmp`
##### Print the target operating system ID string
* `targetos` or `tos`
##### Print report about licenses and EULAs of installed packages
* `licenses`
##### Download rpms specified on the commandline to a local directory
* `download`
##### Download source rpms for all installed packages to a local directory
* `source-download`
##### Check if the needs-reboot flag was set
* `needs-rebooting`
##### List running processes which might still use files and libraries deleted by recent upgrades
* `ps`

!!! example
    ``# zypper ps``

    ``$ sudo zypper ps``
##### Remove old kernels
* `purge-kernels`
### Global Options

???+ info "Show Options"
    | Option | Description | Alias |
    | --- | --- | --- |
    | --color | Use colors in output |
    | --config &lt;FILE&gt;| Use specified config file instead of the default | -c &lt;FILE&gt; |
    | --help | Help | -h |
    | --ignore-unknown | Ignore unknown packages | -i |
    | --no-abbrev | Do not abbreviate text in tables | -A |
    | --no-color | Do no use colors in output |
    | --non-interactive | Do not ask anything, use default answers automatically | -n |
    | --promptids | Output a list of zypper's user prompts |
    | --quiet | Suppress normal output, print only error messages | -q |
    | --table-style &lt;INTEGER&gt; | Table style (0-11) | -s &lt;INTEGER&gt; |
    | --terse | Terse output for machine consumption | -t |
    | --userdata &lt;STRING&gt; | User defined transaction id used in history and plugins |
    | --verbose | Increase verbosity | -v |
    | --version | Output the version number | -V |
    | --xmlout | Switch to XML output | -x |

### Repository Options

???+ info "Show Options"
    | Option | Description | Alias |
    | --- | --- | --- |
    | --disable-repositories | Do not read meta-data from repositories |
    | --gpg-auto-import-keys | Automatically trust and import new repository signing keys |
    | --no-gpg-checks | Ingore GPG check failures |
    | --no-cd | Ignore CD/DVD repositories |
    | --no-refresh | Do not refresh the repositories |
    | --no-remote | Ignore remote repositories |
    | --plus-content &lt;TAG&gt; | Additionally use disabled repositories providing a specific keyword |
    | --plus-repo &lt;URI&gt; | Use an additional repository | -p &lt;URI&gt; |
    | --releasever | Set the value of $releasever in all .repo files |

### Command Options

!!! info
    Command-specific options can be obtained using `# zypper help <COMMAND>`

???+ Example
    ```
    # zypper help in
    install (in) [OPTIONS] <CAPABILITY|RPM_FILE_URI> ...

    Install packages with specified capabilities or RPM files with specified location. A capability is
    NAME[.ARCH][OP<VERSION>], where OP is one of <, <=, =, >=, >.

    Command options:

    -t, --type <TYPE>           Type of package (package, patch, pattern, product).
    -n, --name                  Select packages by plain name, not by capability. Default: false
    -C, --capability            Select packages solely by capability. Default: false
        --details               Show the detailed installation summary. Default: false
        --from <ALIAS|#|URI>    Select packages from the specified repository.
        --oldpackage            Allow to replace a newer item with an older one. Handy if you are doing
                                a rollback. Unlike --force it will not enforce a reinstall. Default:
                                false
        --allow-unsigned-rpm    Silently install unsigned rpm packages given as commandline parameters.
                                Default: false
    -f, --force                 Install even if the item is already installed (reinstall), downgraded or
                                changes vendor or architecture. Default: false
    -r, --repo <ALIAS|#|URI>    Work only with the specified repository.
    -y, --no-confirm            Don't require user interaction. Alias for the --non-interactive global
                                option. Default: false
    -D, --dry-run               Don't change anything, just report what would be done. A meaningful file
                                conflict check can only be performed if used together with
                                '--download-only'. Default: false
        --replacefiles          Install the packages even if they replace files from other, already
                                installed, packages. Default is to treat file conflicts as an error.
                                --download-as-needed disables the fileconflict check. Default: false
    -l, --auto-agree-with-licenses
                                Automatically say 'yes' to third party license confirmation prompt. See
                                'man zypper' for more details. Default: false
        --download <MODE>       Set the download-install mode. Available modes: only, in-advance,
                                in-heaps, as-needed Default: DownloadDefault
    -d, --download-only         Only download the packages, do not install.

    Solver options

        --debug-solver          Create a solver test case for debugging.
        --force-resolution      Force the solver to find a solution (even an aggressive one) rather than
                                asking.
    -R, --no-force-resolution   Do not force the solver to find a solution, let it ask.
        --solver-focus <MODE>   Set the solvers general attitude when resolving a job.
        --recommends            Install also recommended packages in addition to the required ones.
        --no-recommends         Do not install recommended packages, only required ones.

    Expert options

        --allow-downgrade
        --no-allow-downgrade    Whether to allow downgrading installed resolvables.
        --allow-name-change
        --no-allow-name-change  Whether to allow changing the names of installed resolvables.
        --allow-arch-change
        --no-allow-arch-change  Whether to allow changing the architecture of installed resolvables.
        --allow-vendor-change
        --no-allow-vendor-change
                                Whether to allow changing the vendor of installed resolvables.
    ```
