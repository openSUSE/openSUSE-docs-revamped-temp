# Obtaining software

## Introduction

When it comes to getting software, modern Linux operating systems have access to a wide array of methods and formats.
The openSUSE distributions are no exception:

*   The official and unofficial repositories, which you interact with from `zypper`, and which provide binaries packaged as RPM files.

*   Third-party installers, which you interact with from a command prompt by downloading files (for example, with `wget`, `curl`, or `git`).
    These may provide you with a something ready to install, complete with binaries and installation scripts.
    Alternatively, they may only include source code and build scripts, so you must compile them yourself.

*   _Flatpak_ is a delivery tool for containerized desktop application.
    It provides apps packaged up with all their dependencies.
    You typically interact with it using the `flatpak` command-line utility.

*   _Snapcraft_ is Ubuntu's equivalent to Flatpak.
    It similarly provides apps in single files, containerized with their dependencies.
    You typically interact with it using the `snapd` command-line utility.

*   _AppImages_ are a vendor-neutral, cross-distro app format.
    Apps are delivered as a single files, which you manually download from websites and various hosts.
    AppImages do not need any supporting tools, they only need to be marked executable.

The goal of this document is to help you understand these formats and methods, as well as their strengths and weaknesses.
We will also make a few openSUSE-specific recommendations along the way.


## Official and third-party RPM repositories

RPM packages (`.rpm`) are the traditional and most common way of obtaining new software on openSUSE distributions.
Installing RPM packages works like this:

1.  Find the repository providing the package you are looking for.
    *   If the package is provided by one of the official repositories that openSUSE already knows about, such as [download.opensuse.org/update/tumbleweed/](https://download.opensuse.org/update/tumbleweed/), it will show up in `zypper search <package_name>`.
    *   If the package is _not_ provided by an official repository, you will first need to add the repository.

    !!! tip
        Use `zypper lr -d` to list all your repositories and their addresses.

2.   Install the target package with `sudo zypper in <package_name>`.


### The _openSUSE Software_ website

The openSUSE project offers a website where contributors can find software packages for easy installation, especially those built using the _openSUSE Build Service_.
The site can be found at [software.opensuse.org](https://software.opensuse.org/).

It offers a search engine and references to official and unofficial software repositories.
In addition, in some cases, the tool also offers downloadable packages for Debian, Ubuntu, Fedora, or other Linux distributions.

It also offers easy installation using the _YaST_ administration tool of openSUSE.
If a package is available for your version of openSUSE, click the link with the version number.
This will download a `.ymp` file which your web browser should offer to open using the YaST software management tool.
If not, you can click on the `.ymp` file in your file manager and YaST should open it.
YaST will then add any necessary repositories, asking you for permission in the process, and then install the program and its dependencies.

For free and open-source software, using this is usually easier than manually adding a third-party repository.
In some cases, you may also find packages of proprietary freeware.
However, if you need a proprietary application, such as Google Chrome, Skype, or Microsoft Teams, you should obtain it directly from the vendor.
If they have a Linux repository, you can add it to your system, or download and install an RPM directly.

!!! important
    After installing from a `.ymp` file, you will remain subscribed to all RPM repositories referenced in that file.
    In many cases, the package you wanted to install initially isn't the only RPM provided by the repository.
    Carelessly updating (with `zypper up` or `zypper dup`) afterwards can lead to surprising changes in your software selection.
    This is especially when you accidentally subscribe to repositories meant for different distribution versions.


### Using repositories from third-party providers

Certain application providers offer RPM packages and RPM repositories themselves.
In some cases, the RPM package will not only install the desired application but also add the RPM repository to the system, enabling application updates.
For example, Google provides the official Chrome package this way.

1.  After downloading an RPM package, check it.
    Most trustworthy providers cryptographically sign their RPMs, providing you with an `.asc` file containing a public key.
    This lets you verify that the package has not been tampered with:

    ```
    $   rpm --import <the-package-public-key>.asc
    $   rpm --checksig <the-rpm-package-file>.rpm
    ```

2.  To install the package, point `zypper` to the RPM file:

    ```
    sudo zypper in ~/Downloads/<package>.rpm
    ```

We recommend that you enable the _refresh_ setting for the repositories of the applications you use most.
If you don't do this, in some configurations, `sudo zypper dup` will not automatically acquire updates for the package provided by that repository.

1.  Find out the number of of the repo and whether the _refresh_ setting is enabled:

    ```
    $   sudo zypper modifyrepo --refresh <repo_number>
    ```

    The first column of the table, `#`, indicates the repo number.
    The last column of the table, `Refresh`, indicates whether the _refresh_ setting is enabled (`Yes`) or not (`No`).

2.  For repos that are not yet set to refresh automatically, enable the _refresh_ setting:

    ```
    $   sudo zypper modifyrepo --refresh <repo_number>
    ```



### Pros, cons, when to use

__Pros:__

*   The most "natural" way to install programs in the openSUSE ecosystem.

*   Has the best performance (startup time and execution speed) of all solutions.

*   Uses the lowest amount of RAM and hard drive capacity.

__Cons:__

*   Heavily depends upon the libraries and other dependencies provided by your operating system.
    You may not be able to install the latest versions of some programs, because the supporting libraries provided by your distro are too old.
    Conversely, it can also mean that drastic changes in your system-wide dependencies might prevent the program from running at all.

*   Depends on the availability of the distribution packagers and maintainers
    Other solutions are more developer-driven or benefit from cross-distribution maintainership (_Flatpaks_, _Snaps_ and _AppImages_).

*   You depend on the maintainers and software-packagers of your distribution.
    Other solutions are more developer-driven or benefit from cross-distribution maintenance (_Flatpaks_, _Snaps_ and _AppImages_).

__Best used when:__

*   The software you are installing is a core utility or library (e.g. `curl`, `gcc`, `python-devel`).

*   You know that you won't need several versions of the software installed at the same time.

*   You need stability and minimal resource usage.

*   You can afford a marginally slower update pace than with other solutions.


## Third-party remote providers

We use "third-party remote providers" to mean websites, services or remote machines offering automatic installation using scripts provided along with the desired application.

Since the details of the installation depend on custom-made installation scripts, your experience with this method can vary significantly from one provider to another.
Thoroughly follow the instructions mentioned by the provider.
For example, installing the Rust developer stack is a simple as copying and pasting this command into a terminal window:

```
$   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Then follow the interactive instructions.

!!! warning
    Installation scripts might require `sudo` rights.
    Be mindful about what you are installing, and from whom and where you obtained it.


### Pros, cons, when

__Pros:__

*   Has the best performance (startup time and execution speed) of all solutions.

*   Uses the lowest amount of RAM and hard drive capacity.

__Cons:__

*   No control over the behaviour of the script, especially if the script asks to be run as `sudo`.
    This can incur some risk.

*   Heavily depends upon libraries and other dependencies provided by your operating system.
    This means that drastic changes in your system-wide dependencies might prevent the program from running at all.

__Best used when:__

*   You are 100% sure you can trust the provider.

*   The software you are installing is a command-line tool that is not integral to the system.

*   You know that you won't need several versions of the software installed at the same time.

*   You need maximal performance/minimal usage.


## Flatpaks

_Flatpak_ is a complete solution for installing, updating and running desktop applications from a containerized environment.
It makes use of the following components:

*   The `flatpak` command line utility

*   A remote application store at [flathub.org](https://flathub.org/)

*   Application references, such as `com.spotify.Client`, organized by the Flathub store

*   A local repository used to store applications along with their dependencies (by default, `/var/lib/flatpak`)

On openSUSE distributions, you can install support for Flatpak as follows:

1.  Install the command line utility (not required if it's installed already; to double-check, see if `flatpak --version` returns something):
    ```
    $   sudo zypper in flatpak
    ```

2.  Add the Flathub store as an external flatpak repository:
    ```
    $   flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
    ```

3.  Restart your system.
    From now on you will be able to install any of the numerous apps provided from _Flathub_, as in:
    ```
    $   flatpak install --user flathub com.spotify.Client
    ```

Bear in mind:

*   We recommend using the `--user` flag during the installation of Flatpaks.
    With the `--user` flag, applications are installed in your home directory.
    In combination with a separate `/home` partition, this means that applications will remain installed even when the root file system (`/`) is deleted during a new installation.

*   Installing applications (with or without `--user`) should automatically export a `.desktop` file to `/var/lib/flatpak/exports/share/applications` and add the corresponding symlinks to your system.
    This means applications will show up in your desktop environment's application launchers and other menus right after you have installed them (no reboot required).

To update Flatpaks, use:
```
$   flatpak update
```

To update a single Flatpak application, instead do:
```
$   flatpak update <the.application.name>
```


### Additional details

#### How it works

The two main themes behind Flatpaks are _binary versioning control_ and _containerization_.

Binary versioning control (BVC) ensures that the system sees Flatpak applications not as files with a given name.
It also that tracks them against the context of a full versioning control system, such as Git.
Flatpak applications are thus much like Git branches, in that they "wear their origin upon their sleeves".
For example, for the Spotify application, the `flatpak` command has the following tracking information:
```
$   flatpak list
Spotify com.spotify.Client      1.1.55.498.gf9a83c60    stable  system
```

This amounts to, in this order:

*   A human-friendly name

*   The exact name as defined by the provider

*   The (Git) snapshot it is built from

*   The (Git) branch hosting the snapshot (here, _stable_)

*   The installation site (here, it is installed for the _system_, which means it can be run by any other user of the machine).

Under this constraint, any difference in the name or snapshot or branch or installation site will determine a different Flatpak application reference.
Two applications that are byte-for-byte identical will not collide with one another if they are different in other aspects of their reference.
This means that, at any point, you can identify the portion of the codebase from which the application stems and that is relevant to the binary you are running.
This also means you can install multiple variants of one application through multiple application references side by side.

The other main theme behind Flatpaks is containerization.
Here, that means that Flatpak applications run in isolation from one another.
This is made possible by taking advantage of the binary versioning control system mentioned above.
Flatpak allows developers to specify a particular _runtime_ for their app, which is the context against which their applications is to run, and that serves as a proxy for the full operating system.
Because of BVC, developers are free to specify different runtimes for the same byte-to-byte identical binaries, or conversely, the same runtime for widely different binaries, should they happen to have similar requirements.
The Flatpak system then makes sure that runtimes meet the requirements of the applications they are associated with by their developers, and are executing in mutually inaccessible memory spaces.

!!! info
    While Flatpak applications share some dependencies if they share the same runtime, users can't ensure that _all_ dependencies are shared between different applications.
     For users, there is also no way to ensure that different Flatpak applications use the same runtime, as dependencies on runtimes are specified by developers.
    This means that some duplication is to be expected, wasting some disk space.


#### Improving desktop integration (KDE Plasma)

The degree of independence that Flatpaks enjoy with respect to the host operating system means that some frictions may occur now and then.
This is particularly true for applications (and runtimes) which do not play nice with recent rendering and compositing frameworks.
For example, dragging and dropping a file for sending via a messaging application running as a Flatpak might not work as seamlessly as expected.
For these corner cases, you can tinker with the runtime's parameters.

Consider for example `org.telegram.desktop`.
Should your version of this app seem to be affected by the above bug, you can try to run the application as:
```
$   flatpak run --env=QT_QPA_PLATFORM=xcb org.telegram.desktop
```

This instructs the runtime to use X11 instead of Wayland as the compositing client.
There are many other environment parameters.
To learn which to apply to a particular application, read the documentation and GitHub Issues for the Flatpak wrapper of the application.

!!! info
    All Flatpak applications hosted on Flathub have a public repository (often on GitHub) where you can read GitHub Issues and participate in the conversation.
    Do not hesitate to go there now and then, as the tips you might learn are likely to serve you for many other Flatpak applications.

When you are satisfied with the results, you can attach a runtime setting to an application permanently to the current user as in:
```
$   flatpak override --user --env=QT_QPA_PLATFORM=xcb org.telegram.desktop
```

(Remove the `--user` flag to apply the override system-wide.)

This will ensure that the application is always run with the corresponding environment parameter.
More on the override option in the [official Flatpak documentation](https://docs.flatpak.org/en/latest/flatpak-command-reference.html?highlight=override#flatpak-override).


#### Flatseal

Flatseal allows changing Flatpak isolation parameters for applications.
Please refer to Flatseal's [official documentation](https://github.com/tchx84/Flatseal/blob/master/DOCUMENTATION.md).


### Pros, cons, when to use

__Pros:__

*   Decent performance (it runs as fast as "bare" binaries and launches almost as fast)

*   Excellent safety for your operating system (because most dependencies are not shared with the host operating system and because Flatpaks installed with `--user` survive system reinstallations/restorations)

*   Easy to update (`flatpak update`)

__Cons:__

*   Eats up significantly more hard drive space than the alternative methods and formats (because not all dependencies can be shared between Flatpaks)

*   Sometimes rough around the edges in terms of integration with the desktop environment

__Best used when:__

*   You are using many applications with a graphical interface and you don't want to expose them or your system to conflicts of dependencies (useful for Tumbleweed and MicroOS, whose core packages change frequently).

*   You want to have freshly updated and widely tested applications (the same as most other Linux distributions).

*   You plan on using many user applications (the more Flatpaks you have, the more dependencies they can share).


## Snaps

!!! info
    Please help finish this section.

<!-- ### Pros, cons, when -->


## AppImage

AppImage is the simplest cross-platform application package format.
However, you need to do a little manual work.

To install an AppImage, download it and marking it _executable_ with:

```
chmod +x <application_name>.AppImage
```

To run the app, double-click it in your file manager, or run it from your terminal:
```
./<application_name>.AppImage
```

To improve organization of your AppImages, create a dedicated directory for them in your home directory and move each file.
The authors of AppImage recommend `~/Applications` for this purpose.


With AppImages, it is your responsibility to keep the app up-to-date.
While some apps notify you when a new version becomes available, you will usually have to perform the following manually:

*   Download the new version, move it into your desired folder and mark it executable
*   If you have created any shortcuts or launchers, you will have to edit their settings to point to the new version.
*   When everything works, delete the `.AppImage` file of the old version.


### Pros, cons, when

__Pros__:

*   AppImages need no supporting libraries, tooling, or commands.

*   You can install, update, and remove them even without admin privileges on the computer.

*   You can easily run multiple versions side-by-side, as version numbers are typically embedded in the filename.

__Cons__:

*   Maintenance is your responsibility:
    You have to update your apps yourself, manually.

*   Some apps integrate well with the system, but this is not guaranteed.
    Launchers, file associations, etc. may have to be created by hand.

*   Updating the app means manually changing launcher properties, creating new shortcuts, and so on.

*   Making AppImages available to all users on a computer may take extra work.

*   The AppImage format does not offer any kind of isolation or separation between apps.

__Best used when:__

*   The app is not available in RPM form, or the RPM assumes a different OS (such as Fedora) and will not install correctly.

*   There is no Flatpak or Snap, or you don't have or don't want, the extra plumbing of Flatpak or Snap.

*   You don't have privileges to install new software system-wide.


## Summary and conclusion

Even though Leap and Tumbleweed's official repositories are numerous, you will find yourself in search of alternatives at times.
Here are some simple rules of thumbs that emerge from the pros and cons discussed above.

Installing Snaps, Flatpaks, or AppImages is a serious alternative to installing from official repositories if you are looking for a program which:

*   Has a graphical interface

*   Updates often

*   Relies on dependencies that are not provided in official repositories

*   Relies on dependencies that are more recent than those in official repositories

This applies to both Leap and Tumbleweed's official repositories.

When wondering which to use, between Snaps, Flatpaks and AppImages:

*   Favour __Flatpaks__ if you:

    *   Want a streamlined experience or precise maintenance options via the command line

    *   If you plan on using several other Flatpaks.
        The more Flatpaks using the same dependencies there are on your system, the more storage-efficient the format becomes.

*   Favour __AppImages__ if you need a handful of "fire-and-forget" or "portable" (no installation needed) applications.


## References

For a comparison between AppImage, Flatpak and Snap, see [the AppImageKit wiki](https://github.com/AppImage/AppImageKit/wiki/Similar-projects#comparison).
