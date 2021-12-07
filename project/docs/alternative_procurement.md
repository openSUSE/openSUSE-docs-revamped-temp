# Obtaining software

## Introduction

When it comes to getting software, modern Linux operating systems have access to a wide array of methods and formats. The openSUSE distributions are no exception:

* the official and unofficial repositories, which you interact with from zypper, and which provide binaries packaged as RPMs files
* third-party providers, which you interact with from a command prompt through `wget` or `curl` or `git`, and which either provide you with a complete installation solution, including binaries and installation scripts, or include just the source files along with build scripts
* _flatpaks_, which you typically interact with from the `flatpak` command line utility, providing you containerized applications along with all their dependencies
* _Ubuntu snaps_, which you install from the `snapd` command line utility, providing you (just like flatpaks) containerized applications with their dependencies
* _AppImages_, which you manually download from websites and various hosts, providing you with a single executable file which your system will treat as an application, much like what you would see in a macOS environment

The goal of this document is to help you understand these various formats and methods, as well as their strong and weak points, so that you can seamlessly include them in your daily usage. Assuming you are using an openSUSE distribution, we will make a few recommendations specific to these distributions along the way.

## Official and third-party repositories (.rpm)

`.rpm` packages are the traditional and most common way of obtaining new software on openSUSE distributions. You are likely already familiar with the method for obtaining these packages, but to be sure:

1. Find the repository providing the package you are looking for.
    * If the package is provided by an official repository, such as http://download.opensuse.org/update/tumbleweed/, (use `zypper lr -d` to show the exact endpoint of all your repositories), then the desired package will show up in `zypper search <package_name>`.
    * If the package is _not_ provided by an official repository, you will need to first add the repository.
2. Install the target package with `sudo zypper in <package_name>`. In certain cases you won't know the exact name of the package, so after adding the corresponding repository you can simply do `sudo zypper dup` to make sure that the package is installed among all the pending updates.

### Detailed instructions for external repositories

Certain application providers provide you with an `.rpm` file that will automatically add the required repository and install the desired application. When so, you can simply point __zypper__ to the said file and install from there, as in:

```
cd ~/Downloads
sudo zypper in <package>.rpm
```

When doing so make sure the package is safe and the provider trustworthy. Most trustworthy providers sign their `.rpm`s with cryptographic keys, providing you with an `.asc` file containing a public key. You can the use the public key to verify the package, as in:

```
$   rpm --import <the-package-public-key>.asc
$   rpm --checksig <the-rpm-package-file>.rpm
```

We recommend making sure that third-party repositories for the applications you use the most be added with the __refresh__ setting enabled, as in:

```
$   sudo zypper addrepo --refresh <host_address>
```

On some configurations not doing so might result in `sudo zypper dup` not automatically acquiring new updates for the package provided by that repository. You can make sure your repositories __refresh__ automatically by listing them and paying attention at the (r) column -- the last -- obtained from `zypper lr`. 

### Pros, cons, when

__Pros__:

* the most 'natural' way to install programs in the openSUSE ecosystem
* has the best performance (startup time + execution speed) of all solutions
* has the smallest footprint on your RAM and hard drive

__Cons__:

* heavily depends upon the libraries and other dependencies provided by your operating system, which means that drastic changes in your system-wide dependencies might prevent the program from running at all
* depends on the availability of the distribution's packagers and maintainers, as opposed to solutions "closer" to the developers or that can benefit from cross-distribution maintainership (_flatpaks_, _AppImages_, _Ubuntu snaps_)

__Go for it when__:

* you are aiming for core utilities or libraries (e.g. _curl_, _gcc_, _python-devel_) and you know beforehand that you won't need to have several versions of these installed at the same time
* you need maximal performance / minimal footprint
* you can afford a marginally slower update pace than with other solutions

## Third-party remote providers

We call _third-party remote providers_ as a catch-all term for websites, services or remote machines offering you an automatic installation solution provided by installation scripts provided along with the desired application.

Since all the details for the installation depends on custom made installation scripts, your experience with this method can vary significantly from one provider to another. Here your best bet is to thoroughly follow the instructions mentioned by the provider. For example, installing the Rust developer stack is a simple as copy-pasting into your command line the line:

```
$   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

and following the interactive instructions.

!!! warning
    Installation scripts might require sudo rights, so be mindful about what you are installing, and who you are installing from. 

### Pros, cons, when

__Pros__:

* has the best performance (startup time + execution speed) of all solutions
* has the smallest footprint on your RAM and hard drive

__Cons__:

* no control over the behaviour of the script -- especially if the script asks to be run as `sudo` -- which may incur some risk 
* heavily depends upon the libraries and other dependencies provided by your operating system, which means that drastic changes in your system-wide dependencies might prevent the program from running at all
* depends on the availability of the distribution's packagers and maintainers, as opposed to solutions "closer" to the developers or that can benefit from cross-distribution maintainership (_flatpaks_, _AppImages_, _Ubuntu snaps_)

__Go for it when__:

* you are 100% sure you can trust the provider
* you are aiming for core utilities or libraries (e.g. _curl_, _gcc_, _python-devel_) and you know beforehand that you won't need to have several versions of these installed at the same time
* you need maximal performance / minimal footprint
* you can afford a marginally slower update pace than with other solutions

## Flatpaks

_Flatpak_ is a complete solution for installing, updating and running desktop applications from a containerized environment. It makes use of the following components:

* the `flatpak` command line utility
* a remote application store at https://flathub.org/home
* application references, such as `com.spotify.Client`, organized by the flathub store
* a local repository used to store applications along with their dependencies (by default, `/var/lib/flatpak`)

On openSUSE distributions, you can install flatpak as follows:

Install the command line utility (not required if it's installed already; to double-check see if `flatpak --version` returns something):
```
$   sudo zypper in flatpak
```

Add the flathub store as flatpak external repository:
```
$   flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

And finally restart your system. From now on you will be able to install any of the very numerous apps provided from _flathub_, as in:
```
$   flatpak install --user flathub com.spotify.Client
```

Two things are important to bear in mind:

1. Using the `--user` flag is recommended for this reason: if the user has a separate `/home` and installs flatpaks using the `--user` flag, these applications will remain even after the entire file system is reinstalled, which means that the installed flatpaks can be reused after restoring the system.
2. Installing applications (with or without `--user`) should automatically export a `.desktop` file to `/var/lib/flatpak/exports/share/applications` and add the corresponding symlinks to your system. This means applications will show up in your desktop environment's application launchers and other menus right after you have installed them (no reboot required).

Updating your flatpaks is then as easy as:
```
$   flatpak update
```

To update a single flatpak application, do instead:
```
$   flatpak update <the.application.name>
```

### Additional details

#### How it works

The two main themes behind flatpaks are _binary versioning control_ and _containerization_.

Binary versioning control (BVC) ensures that the system sees flatpak applications not just as a bunch of files with a given name, but in a way that tracks them against the context of a full versioning control system, such as Git. Flatpak applications are thus much like Git branches, in that they "wear their origin up their sleeves". For example, the complete reference of the Spotify application on my machine as of writing this is:
```
$   flatpak list
Spotify com.spotify.Client      1.1.55.498.gf9a83c60    stable  system
```

which amounts to, in this order: a human-friendly name, the exact name as fixed by the provider, the (git) snapshot it is built from, the (git) branch hosting the snapshot (here _stable_), as well as the installation site on my machine (here, it is installed for the _system_, which means it can be run by any other user on my machine.)

Under this constraint, any difference in the name or snapshot or branch or installation site will determine a different flatpak application reference.  Two applications that are byte-for-byte identical will not collide with one another if they are different under other aspects of their reference. This means that, at any point, you can identify the portion of the codebase from which the application stems and that is relevant to the binary you are running. This also means you can have multiple versions of "the same" application through multiple application references.

The other main theme behind flatpaks is containerization. Here it means that flatpak applications run in isolation from one another. This is made possible by taking advantage of the binary versioning control system mentioned above. Indeed, flatpak allows developers to specify a particular _runtime_ for their app, which is the context against which their applications is to run, and that serves as a proxy for the full operating system. Because of BVC developers are free to specify different runtimes for the same byte-to-byte identical binaries, or conversely, the same runtime for widely different binaries, should they happen to have similar requirements. The flatpak system then makes sure that runtimes meet the requirements of the applications they are associated with by their developers, and are executing in mutually inaccessible memory spaces.

!!! info
    Even though the sharing of runtimes means that flatpak applications will share some of their dependencies, there is no reliable way of ensuring that all their dependencies are thus shared, or that a developer has specified the same runtime as the one specified by the developers of the other flatpak applications installed by the user. This means that some duplication is to be expected, wasting some disk space.

#### Improving desktop integration (KDE Plasma)

The degree of independence that flatpaks enjoy with respect to the host operating system means that some frictions may occur now and then. This is particularly true for applications (or for runtimes) which do not always play nice with recent rendering and compositing frameworks. For example, drag-dropping a file for sending via a messaging application run as flatpak might not work as seamlessly as expected. For these corner cases, there is not much choice but to tinker with the runtime's parameters.

Consider for example `org.telegram.desktop`. Should your version of this app seem to be affected by the above bug, you can try to run the application as:
```
$   flatpak run --env=QT_QPA_PLATFORM=xcb org.telegram.desktop
```

which basically instructs the runtime to use X11 instead of Wayland as the compositing client. They are countless other environment parameters you can use, and the right approach is usually to read the documentation and Git Issues of your favorite flatpak applications.

!!! info
    Every single flatpak has is a public repository (often on GitHub) where you read Issues and participate in the conversation. Do not hesitate to go there now and then, as the tips you might learn are likely to serve you for many other flatpak applications.

When you are satisfied with the results, you can attach a runtime setting to an application permanently to the current user as in:
```
$   flatpak override --user --env=QT_QPA_PLATFORM=xcb org.telegram.desktop
```

(Remove the --user flag to apply the override system-wide.)

This will ensure that the application is always run with the corresponding environment parameter. More on the override option in the [official Flatpak documentation](https://docs.flatpak.org/en/latest/flatpak-command-reference.html?highlight=override#flatpak-override).

#### Flatseal
Please refer to Flatseal's [official documentation](https://github.com/tchx84/Flatseal/blob/master/DOCUMENTATION.md).

### Pros, cons, when

__Pros__:

* decent performance (it runs as fast as "bare" binaries and launches almost as fast)
* excellent safety for your operating system (because most dependencies are not shared with the host operating system and because flatpaks installed with `--user` survive system reinstallations/restorations)
* easy to update (`flatpak update`)

__Cons__:

* eats up significantly more hard drive space than the alternative methods and formats (because not all dependencies can be shared between flatpaks)
* sometimes rough around the edges in terms of integration with the desktop environment

__Go for it when__:

* you are using many applications with a graphical interface and you don't want to expose them or your system to conflicts of dependencies (useful for Tumbleweed and microOS, whose core packages change frequently)
* you want to have freshly updated and widely tested applications (the same as most other Linux distributions)
* you plan on using many user applications (the more flatpaks you have, the more dependencies they can share)

## Snaps

!!! warning
    Please help finish this section.

### Pros, cons, when

!!! warning
    Please help finish this section.

## AppImages

!!! warning
    Please help finish this section.

### Pros, cons, when

!!! warning
    Please help finish this section.

## Summary and conclusion
Even though Leap and Tumbleweed's official repositories are numerous, you will find yourself in search of alternatives at times. Here are some simple rules of thumbs that emerge from the Pros and cons discussed above:

If you are looking for a program with a graphical interface that...

1. updates often; or 
2. relies on very recent dependencies; or
3. requires you to install dependencies that:
    * are provided by the official repositories; but 
    * differ in version from those provided by the official repositories; then
    you will be better off using ; then snaps, flapaks or appimages offer you serious alternatives to official repositories, which may update less frequently. This applies to both Leap and Tumbleweed's official repositories.

When wondering which to use, between snaps, flatpaks and appimages:

1. favour __flatpaks__ if you want a streamlined experience or precise maintenance options via the command line or if you plan on using several other flatpaks (the more flatpaks, the most cost-efficient the sacrifice of storage space they demand)
2. favour __appimages__ if you need a handful of "fire-and-forget" or "portable" (no installation needed) applications.

## References
* Comparison between AppImage, Flatpak and Ubuntu Snap: https://github.com/AppImage/AppImageKit/wiki/Similar-projects#comparison
