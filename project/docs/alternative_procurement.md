# Obtaining software

## Introduction

When it comes at getting software, modern Linux operating systems have access to a wide array of methods and formats. The openSUSE distributions are no exception:

* the official and unofficial repositories, which you interact with from `zypper`, and which provide binaries packaged as RPM files.
* third-party providers, which you interact with from a command prompt by downloading files (for example, with `wget`, `curl`, or `git`). These may provide you with a something ready to install, complete with binaries and installation scripts, or may include just source code and build scripts, so you must compile them yourself.
* _Flatpak_ is the GNOME Project's containerized application delivery tool. You typically interact with it using the `flatpak` command, and it provides apps packaged up with all their dependencies.
* _Snapcraft_ is Ubuntu's equivalent to Flatpal, and similarly, it provides apps in single files, containerized with their dependencies. You typically install from the `snapd` command line utility.
* _AppImages_ are a vendor-neutral cross-distro app format. Apps are delivered as a single files, which you manually download from websites and various hosts. No supporting tools are needed; it is enough to mark the file as executable with `chmod +x <filename>`, then your computer will treat it as an application, much like what you would see in a macOS environment.

The goal of this document is to help you understand these various formats and methods, as well as their strengths and weaknesses, so that you can include them in your daily usage. We will also make a few openSUSE-specific recommendations along the way.

## Official and third-party RPM repositories

`.rpm` packages are the traditional and most common way of obtaining new software on openSUSE distributions. Just in case you are not already familiar with how these work:

1. Find the repository providing the package you are looking for.
    * If the package is provided by one of the  official repositories that openSUSE already knows about, such as http://download.opensuse.org/update/tumbleweed/, then the desired package will show up in `zypper search <package_name>`.
    * If the package is _not_ provided by an official repository, you will need to first add the repository.
> :point_right: **TIP:** **Listing repositories**
>
> Use `zypper lr -d` to list all your repositories and their addresses.
2. Install the target package with `sudo zypper in <package_name>`. In certain cases you won't know the exact name of the package, so after adding the corresponding repository you can simply do `sudo zypper dup` to make sure that the package is installed among all the pending updates.

## The _openSUSE Software_ website

The openSUSE project maintains a website where contributors can host software packages for easy installation, especially those built using SUSE's _Open Build Service_. The site can be found at:

[openSUSE Software](https://software.opensuse.org/) — https://software.opensuse.org

It offers a search engine and easy one-click installation using the openSUSE _YaST_ administration tool, but it also offers packages for multiple other distributions at the discretion of the maintainer. You can find packages for Debian, Ubuntu, Fedora and other distributions as well as openSUSE.

If a package is available for your version of openSUSE, generally, all you need to do is click the link with the version number. This will download a `.ymp` file which your web browser should offer to open using the YaST software management tool. If not, you can click on the `.ymp` file in your file manager and YaST should open it. YaST will then add any necessary repositories, asking you for permission in the process, and then install the program and its dependencies.

This is generally easier than manually adding a third-party repository, so for FOSS programs, this should probably be the first place you look.

In some cases, you may even find packages of proprietary freeware, but generally, if you need a proprietary Linux program, for example Google Chrome, Skype, or Microsoft Teams, it is preferable to obtain it direct from the vendor. If they have a Linux repository, you can add it to your system, or failing that simply download and install an RPM directly. This is covered in the next section.

### Detailed instructions for external repositories

Certain application providers provide you with an RPM file that will install the desired application and automatically add the required repository, so you get updates from then on. In this case, you can simply point `zypper` to the RPM file:

```
cd ~/Downloads
sudo zypper in <package>.rpm
```

Before you do this, you should make sure the package is safe and the provider trustworthy. Most trustworthy providers crytographically sign their RPMs, providing you with an `.asc` file containing a public key. This lets you verify that the package has not been tampered with, like so:

```
$   rpm --import <the-package-public-key>.asc
$   rpm --checksig <the-rpm-package-file>.rpm
```

We recommend that you enable the _refresh_ setting for the repositories of the applications you use the most, like so:

```
$   sudo zypper addrepo --refresh <host_address>
```

If you don't do this, in some configurations, `sudo zypper dup` will not automatically acquire updates for the package provided by that repository. You can make sure your repositories _refresh_ automatically by listing them with the command `zypper lr`, and checking the last column, headed `(r)`, for each repo.

### Pros, cons, when to use

__Pros__:

* the most 'natural' way to install programs in the openSUSE ecosystem
* has the best performance (startup time + execution speed) of all solutions
* uses the smallest amount of RAM and disk space

__Cons__:

* heavily depends the libraries and other dependencies provided by your operating system. This may mean you are unable to install the latest versions of some programs, because the supporting libraries provided by your distro are too old.
* you are depending on the maintainers and software-packagers of your distribution, as opposed to packaging methods "closer" to the developers or which benefit from cross-distribution maintenance  (such as Flatpaks, Snaps or AppImages).

__Best used when__:

* you are aiming for core utilities or libraries (e.g. _curl_, _gcc_, _python-devel_) and you know beforehand that you won't need to have several versions of these installed at the same time
* you need stability and the minimal resource footprint
* you can afford a marginally slower update pace than with other solutions

## Third-party remote providers

We use "third-party remote providers" as a catch-all term for websites, services or remote machines offering you automatic installation by installation scripts provided along with the desired application.

Since the details of the installation depend on custom-made installation scripts, your experience with this method can vary significantly from one provider to another. Here your best bet is to thoroughly follow the instructions mentioned by the provider. For example, installing the Rust developer stack is a simple as copying and pasting this command into a terminal window:

```
$   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

... and following the interactive instructions.

> :warning: **WARNING: Beware of granting admin rights!**
>
> **Installation scripts might require `sudo` rights, so be mindful about what you are installing, and from whom and where you obtained it.**


### Pros, cons, when

__Pros__:

* has the best performance (startup time + execution speed) of all solutions
* has the smallest footprint on your RAM and hard drive

__Cons__:

* no control over the behaviour of the script -- especially if the script asks to be run as `sudo` -- which may incur some risk
* heavily depends the libraries and other dependencies provided by your operating system, which means that drastic changes in your system-wide dependencies might prevent the program from running at all
* depends on the availability of the distribution's packagers and maintainers, as opposed to solutions "closer" to the developers or that can benefit from cross-distribution maintainership (_flatpaks_, _AppImages_, _Ubuntu snaps_)

__Best used when__:

* you are 100% sure you can trust the provider
* you are aiming for core utilities or libraries (e.g. _curl_, _gcc_, _python-devel_) and you know beforehand that you won't need to have several versions of these installed at the same time
* you need maximal performance / minimal footprint
* you can afford a marginally slower update pace than with other solutions

## Flatpaks

_Flatpak_ is a complete solution for installing, updating and running desktop applications from a containerized environment. It makes use of the following components:

* the `flatpak` command line utility
* a remote application store at https://flathub.org/home
* application references, such as `com.spotify.Client`, organized by the Flathub store
* a local repository used to store applications along with their dependencies (by default, `/var/lib/flatpak`)

On openSUSE distributions, you can install flatpak as follows:

Install the command line utility (not required if it's installed already; to double-check, see if `flatpak --version` returns something):
```
$   sudo zypper in flatpak
```

Add the flathub store as an external flatpak repository:
```
$   flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

And finally restart your system. From now on you will be able to install any of the numerous apps provided from _Flathub_, as in:
```
$   flatpak install --user flathub com.spotify.Client
```

Two things are important to bear in mind:

1. Using the `--user` flag is recommended for this reason: if the user has a separate `/home` and installs flatpaks using the `--user` flag, these applications will remain even after the entire file system is reinstalled, which means that the installed flatpaks can be reused after restoring the system.
2. Installing applications (with or without `--user` should automatically export a `.desktop` file to `/var/lib/flatpak/exports/share/applications` and add the corresponding symlinks to your system. This means applications will show up in your desktop environment's application launchers and other menus right after you have installed them (no reboot required).

Updating your flatpaks is then as easy as:
```
$   flatpak update
```

To update a single flatpak application, instead do:
```
$   flatpak update <the.application.name>
```

### Additional details

#### How it works

The two main themes behind flatpaks are _binary versioning control_ and _containerization_.

Binary versioning control (BVC) ensures that the system sees flatpak applications not just as a bunch of files with a given name, but in a way that tracks them against the context of a full versioning control system, such as Git. Flatpak applications are thus much like Git branches, in that they "wear their origin upon their sleeves". For example, the complete reference of the Spotify application on my machine as of this writing is:
```
$   flatpak list
Spotify com.spotify.Client      1.1.55.498.gf9a83c60    stable  system
```

which amounts to, in this order: a human-friendly name, the exact name as fixed by the provider, the (Git) snapshot it is built from, the (Git) branch hosting the snapshot (here _stable_), as well as the installation site on my machine (here, it is installed for the _system_, which means it can be run by any other user on my machine.)

Under this constraint, any difference in the name or snapshot or branch or installation site will determine a different flatpak application reference.  Two applications that are byte-for-byte identical will not collide with one another if they are different under other aspects of their reference. This means that, at any point, you can identify the portion of the codebase from which the application stems and that is relevant to the binary you are running. This also means you can have multiple versions of "the same" application through multiple application references.

The other main theme behind flatpaks is containerization. Here, that means that flatpak applications run in isolation from one another. This is made possible by taking advantage of the binary versioning control system mentioned above. Indeed, flatpak allows developers to specify a particular _runtime_ for their app, which is the context against which their applications is to run, and that serves as a proxy for the full operating system. Because of BVC, developers are free to specify different runtimes for the same byte-to-byte identical binaries, or conversely, the same runtime for widely different binaries, should they happen to have similar requirements. The flatpak system then makes sure that runtimes meet the requirements of the applications they are associated with by their developers, and are executing in mutually inaccessible memory spaces.

> :exclamation: **NOTE: Shared dependencies**
>
> Even though the sharing of runtimes means that flatpak applications will share some of their dependencies, there is no reliable way of ensuring that _all_ their dependencies are thus shared, or that a developer has specified the same runtime as the one specified by the developers of the other flatpak applications installed by the user. This means that some duplication is to be expected, wasting some disk space.

#### Improving desktop integration (KDE Plasma)

The degree of independence that flatpaks enjoy with respect to the host operating system means that some friction may occur now and then. This is particularly true for applications (or runtimes) which do not always play nice with recent rendering and compositing frameworks. For example, dragging-and-dropping a file for sending via a messaging application that is running from a flatpak might not work as seamlessly as expected. For these corner cases, there is not much choice but to tinker with the runtime's parameters.

Consider for example `org.telegram.desktop`. Should your version of this app seem to be affected by the above bug, you can try to run the application as:
```
$   flatpak run --env=QT_QPA_PLATFORM=xcb org.telegram.desktop
```

which basically instructs the runtime to use X11 instead of Wayland as the compositing client. They are countless other environment parameters you can use, and the right approach is usually to read the documentation and Git Issues of your favorite flatpak applications.

!!! info
    Every single flatpak has is a public repository (often on GitHub) where you read Issues and participate to the conversation. Do not hesitate to go there now and then, as the tips you might learn are likely to serve you for many other flatpak applications.

When you are satisfied with the results, you can attach a runtime setting to an application permanently to the current user as in:
```
$   flatpak override --user --env=QT_QPA_PLATFORM=xcb org.telegram.desktop
```

(Remove the --user flag to apply the override system-wide.)

This will ensure that the application is always run with the corresponding environment parameter. More on the override option in the [official Flatpak documentation](https://docs.flatpak.org/en/latest/flatpak-command-reference.html?highlight=override#flatpak-override).

#### Flatseal
Please refer to Flatseal's [official documentation](https://github.com/tchx84/Flatseal/blob/master/DOCUMENTATION.md).

### Pros, cons, when to use

__Pros__:

* decent performance (it runs as fast as "bare" binaries and launches almost as fast)
* excellent safety for your operating system (because most dependencies are not shared with the host operating system and because flatpaks installed with `--user` survive system reinstallations/restorations)
* easy to update (`flatpak update`)

__Cons__:

* eats up significantly more hard drive space than the alternative methods and formats (because not all dependencies can be shared between flatpaks)
* sometimes rough around the edges in terms of integration with the desktop environment

__Best used when__:

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

AppImage is the simplest cross-platform application package format, but that means that you need to do a little more work.

Installing an AppImage is as simple as downloading it from the project's website and then marking it as executable. You might wish to put it in its own folder, or have a dedicated folder for AppImages. For instance, if you made such a folder in your home directory:

    mkdir ~/appimage

You might download, for example, the Franz messaging application from https://meetfranz.com/. Move it to your new `appimage` folder:

     mv ~/Downloads/Franz-5.7.0.AppImage ~/appimage

Then all you have to do is mark the file as executable:

     chmod +x ~/appimage/Franz-5.7.0.AppImage

And you are done. You can now run the app by double-clicking on it, or by typing in your terminal:

    ~/appimage/Franz-5.7.0.AppImage

(You can probably just type `appimage/F` and then press the Tab key.)

The downside of AppImage is that it becomes your responsibility to keep the app up to date. Some apps may tell you when a new version becomes available, but you will have to go, download it, put it in your desired folder, and mark it as executable. If you have any shortcuts or launchers, you will have to edit their settings to point to the new version, and once you are happy that it works, remove the old version.

### Pros, cons, when

**Pros**:

* AppImages need no supporting libraries, tooling, or commands
* You can install, update, and remove them even without admin privileges on the computer
* You can easily run multiple versions side-by-side, as version numbers are typically embedded in the filename

**Cons**:

* Maintenance is your responsibility: you have to update your apps yourself, by hand.
* Some apps integrate well with the system, but this is not guaranteed. File associations and things may have to be done by hand
* Updating the app means manually changing launcher properties, creating new shortcuts, and so on.
* Making them available to all users on a computer may take extra work.
* The AppImage format does not offer any kind of isolation or separation between apps.

**Best used when**:

* The app is not available in RPM form, or the RPM assumes a different OS (such as Fedora) and will not install correctly
* There is no Flatpak or Snap, or you don't have, or don't want, the extra plumbing of Flatpak or Snap
* You can't or don't have the privileges to install new software


## Summary and conclusion
Even though Leap and Tumbleweed's official repositories are numerous, you will find yourself in search of alternatives at times. Here are some simple rules of thumbs that emerge from the Pros and cons discussed above:

If you are looking for a program with a graphical interface that...

1. updates often; or
2. relies on very recent dependencies; or
3. requires you to install dependencies that:
    * are provided by the official repositories; but
    * differ in version from those provided by the official repositories; then
    you will be better off using ; then snaps, flapaks or appimages offer you serious alternatives to official repositories, which may update less frequently. This applies to both Leap and Tumbleweed's official repositories.

When wondering which to use, between Snaps, Flatpaks and AppImages:

1. favour __flatpaks__ if you want a streamlined experience or precise maintenance options via the command line or if you plan on using several other flatpaks (the more flatpaks, the most cost-efficient the sacrifice of storage space they demand)
2. favour __appimages__ if you need a handful of "fire-and-forget" or "portable" (no installation needed) applications.

## References
* Comparison between AppImage, Flatpak and Ubuntu Snap: https://github.com/AppImage/AppImageKit/wiki/Similar-projects#comparison
