## Setting up MicroOS as a desktop OS

!!! warning 
    Parts of this documentation are not in line with the _official_ intent of the developers such as the use of _snap_ packages which are considered a security risk as proper _snap_ to _system_ separation is not possible. You can also visit the [official documentation](https://en.opensuse.org/Portal:MicroOS) for _upstream_ recommendations. Also the use of _MicroOS Desktop_ with _KDE Plasma_ is discouraged as it is not being actively maintained.

### Audience and context
This document is meant for users interested in using the openSUSE MicroOS as a desktop. Although this operating system is offered in two flavours (one based on Leap, the other on Tumbleweed), we will only be concerned with the Tumbleweed base, as the Leap base does not provide the required patterns for desktop usage.

### Pre-installation

  1. Download the Tumbleweed base ISO image from [https://en.opensuse.org/Portal:MicroOS/Downloads](https://en.opensuse.org/Portal:MicroOS/Downloads)
  2. Write the ISO on a USB drive and start the installation process.
  3. Select GNOME or KDE Plasma as desktop environment.
  4. If desired, select _encryption_. 

In what follows:

  * commands starting with `$` can be copied straight into your terminal prompt.
  * commands starting with `#` can only be run inside a transactional-update shell.

!!! info
    A _transactional-update shell_ is a chroot where you can open the snapshot you want to boot into next. In that context you can make adjustments to the normally immutable root filesystem. It is meant mostly for advanced users, so mind your keystrokes.

### Ways to install applications
You can install applications in several ways:

  * flatpaks from flathub -- the _preferred solution_ 
  * RPM's via a package manager
  * RPM's in a toolbox `toolbox -u`
  * snaps (requires some extra setup to get snapd installed)
  * AppImages

!!! info
    The reason _flatpaks_ are preferred is that they provide a self-contained environment offering a pleasant balance between security, dependence integrity and performance. See [here](https://flatpak.org/) for details.

### Package manager options
MicroOS now has a few different package managers as an option during install:

  * transactional-update with zypper `the old default, and still the default for the server version`
  * Packagekit + tukit `the new default, using pkcon and tukit`
  * microdnf `an experimental package manager based on dnf`

The reason for changing package managers is because pkcon gives the opportunity to use Gnome Software or Discover for KDE as an installer of RPM's. And currently we are working on the ability to let Gnome Software and Discover perform system upgrades.

#### Transcational-update _old default_
commands for transactional-update are:

  * `sudo transactional-update pkg install package_name` install a rpm package
  * `sudo transactional-udpdate dup` perform a system upgrade to the next release
  * `sudo transactional-update shell` open a shell of the next snapshot (you can use zypper commands there)

#### Packagekit + tukit _new default_
commands for Packagekit and tukit:

  * `pkcon install package_name` install a rpm package
  * `pkcon update` perform a system upgrade to the next release
  * `sudo tukit execute bash` open a shell of the next snapshot (you can use zypper commands there)

#### Microdnf
commands for microdnf:

  * `microdnf install package_name` install a rpm package
  * `microdnf upgrade` perform a system upgrade to the next release

### Full drive encryption
If you need full drive encryption you can go through the ISO installer as normal, except for the last screen, where you need to click on _Partitioning_. Then use the guided partitioner to enable encryption. _LVM_ is not necessary for it to work. 

Bear in mind however that with full drive encryption, you will need to enter your password twice: 

  1. the first time to open the grub menu
  2. the second time to decrypt during boot.

Alternatively, the second decrypt can be performed automatically; instructions are provided at [manual](https://en.opensuse.org/SDB:Encrypted_root_file_system).

!!! info
    Full disk encryption is _not_ recommended for SSD's as trimming is currently not supported due to the read only nature of the / filesystem during boot.


### Installation: GNOME
_Gnome is currently in a beta stage._

At first boot flatpaks are enabled and some flatpaks are installed by default (Mozilla Firefox, Gedit and Gnome Calculator).

Make sure to install a browser via transactional-updates if you want to use GNOME extensions. The easiest way to install GNOME extensions is via the website provided by the [GNOME project](https://extensions.gnome.org/). For this to work however, you will need a browser installed on the system via transactional-update, as it will not work inside a containerized environment (like flatpaks, snaps, AppImages or in a toolbox). The browser will need to have the plugin mentioned on the top of the page when you first open it. 

After installation of the plugin, you can reload the page and choose the GNOME Extensions you want to install.

To get GNOME extensions running, you need the following command to install Chromium directly:
```
$ pkcon install chromium
$ sudo reboot
```

### Installation: KDE

  * In _Discover_, enable the _flatpak_ repository.
  * For theming, normal theme installations don’t work in KDE Plasma. But you can install themes via a transactional-update shell:
    * (For example the Arc theme and papirus-icon-theme)
```
$ sudo transactional-update shell
  # cd /tmp
  # wget -qO- https://raw.githubusercontent.com/PapirusDevelopmentTeam/arc-kde/master/install.sh | sh
  # wget -qO- https://git.io/papirus-icon-theme-install | sh
  # exit
$ sudo reboot
```
  * Other themes (also for GNOME):
    * [Qogir-icon-theme](https://github.com/vinceliuice/Qogir-icon-theme)
    * [Matcha-gtk](https://github.com/vinceliuice/Matcha-gtk-theme)
    * [Matcha-KDE](https://github.com/vinceliuice/Matcha-kde)
    * [Akwa-gtk](https://github.com/berkiyo/akwa)
    
### General tips
Updates are now to be initialized manually with:
```
$ sudo transactional-update dup
$ sudo reboot
```
Or
```
$ sudo transactional-update shell
  # zypper dup
  # exit
$ sudo reboot
```
Or
```
$ pkcon update
```
Or
```
$ sudo tukit execute bash
  # zypper dup
  # exit
$ sudo reboot
```
#### Useful extra packages to install

Some useful extra packages to install include:

  * `ntfs-3g ` for NTFS partitions
  * `fuse-exfat` for exfat partitions
  * `nano` instead of vim

You can install both with:
```
$ pkcon install tlp ntfs-3g fuse-exfat nano  
$ sudo reboot
```


### Installing flatpaks

The `flatpak` package is already installed and the flathub repo enabled within a Gnome install. Within KDE Plasma it is possible to enable the flathub repo from within _Discover_. 

Updating flatpaks via the terminal are done with the command `sudo flatpak update`

Some flatpaks you could install from [flathub](https://flathub.org/home):

  - [Firefox](https://flathub.org/apps/details/org.mozilla.firefox)
  - [Libreoffice](https://flathub.org/apps/details/org.libreoffice.LibreOffice)
  - [VLC](https://flathub.org/apps/details/org.videolan.VLC)
  - etc.

### Installing snaps

Snaps are mainly used with Ubuntu, but it is possible to install them as well in openSUSE MicroOS with the following commands.
!!! info
    _It is NOT recommended to install snaps on openSUSE MicroOS, there is a possibility this will break during an update or reboot_ 

First, Snapd must be installed on the system with transactional-update:

```
$ sudo tukit execute bash
  # zypper addrepo --refresh https://download.opensuse.org/repositories/system:/snappy/openSUSE_Tumbleweed snappy
  # zypper --gpg-auto-import-keys refresh
  # zypper ref
  # zypper dup --from snappy
  # zypper in snapd
  # exit
$ sudo reboot
```

After a reboot, snapd will be installed. Files installed from Snapd will be contained in the [/snap directory](https://snapcraft.io/docs/system-snap-directory) which can be created as a subvolume on MicroOS.

```
$ source /etc/profile
$ sudo mksubvolume /snap
```

Ubuntu uses [AppArmor](https://ubuntu.com/server/docs/security-apparmor) to handle permissions of Snap applications. Ensure AppArmor is enabled with `sudo systemctl status apparmor` - if systemd shows AppArmor is stopped, start it now (and on every boot) with `sudo systemctl enable --now apparmor.service`

Once AppArmor is running, Snapd can be started with:

```
$ sudo systemctl enable --now snapd
$ sudo systemctl enable --now snapd.apparmor
```
You can install snaps via the terminal (for example the snap store where you can find snaps and install them via a graphical interface):
```
$ sudo snap install snap-store
```

### Installing virtualbox
```
$ sudo tukit execute bash
  # zypper ref
  # zypper in virtualbox-qt
  # usermod -a -G vboxusers <myuser>
  # exit
$ sudo reboot
```

#### Adding the extension pack for the host
_Installation can be done after you've rebooted into a new snapshot where virtualbox-qt is installed_
```
$ sudo tukit execute bash
  # cd /tmp
  # LatestVirtualBoxVersion=$(wget -qO - https://download.virtualbox.org/virtualbox/LATEST-STABLE.TXT) && wget "https://download.virtualbox.org/virtualbox/${LatestVirtualBoxVersion}/Oracle_VM_VirtualBox_Extension_Pack-${LatestVirtualBoxVersion}.vbox-extpack"
  # VBoxManage extpack install --replace Oracle_VM_VirtualBox_Extension_Pack-${LatestVirtualBoxVersion}.vbox-extpack
  # exit
$ sudo reboot
```

### Adding software from OBS
Find what you need on [https://software.opensuse.org](https://software.opensuse.org). Then choose the openSUSE Tumbleweed repository, as MicroOS desktop is built from the same base as Tumbleweed. Everything that runs on Tumbleweed should in theory run reliably on MicroOS Desktop.

You can't use the 1-click install as there is no YaST and the 1-click installer has no access to a transactional-update. So for example if you install `zerotier`:

  1. Pick a _TumbleWeed_ repository.
  2. Click on _Expert Download_.
  3. Choose the option _Add repository and install manually_.
  4. Copy the commands into a _transactional-update shell_
```
$ sudo tukit execute bash
  # zypper addrepo -cf https://download.opensuse.org/repositories/home:alphard:RHEL/openSUSE_Tumbleweed/home:alphard:RHEL.repo 
  # zypper ref
  # zypper in zerotier-one
  # exit
$ sudo reboot
```
If you have added repositories to `zypper` by yourself, it is possible that they will not refresh automatically with a `dup` command. You will have to give them a flag for autorefresh: (or add the `-cf` to the command after `addrepo` or `ar`)
```
$ sudo tukit execute bash
  # zypper lr -p
  # zypper mr -f <number of repo you want to add Yes to in the Refresh colomn>
  # exit
$ sudo reboot
```

### Tips if you're installing on laptop
Two important packages were found missing for some users:

* `tlp` for better battery management
* `ModemManager` for laptops with a WWAN modem

You can install both with:
```
$ pkcon install tlp ModemManager libmbim usb_modeswitch NetworkManager-connection-editor  
$ sudo reboot
```

### Some useful aliases for bash to type the commands faster
Edit the file `~/.bashrc`

And add the following lines in the file at the end:
```
#My custom aliases
alias shalt="sudo systemctl halt"
alias sboot="sudo systemctl reboot"
alias pu="pkcon update"
alias steb="sudo tukit execute bash"
alias sts="sudo transactional-update shell"
alias stsc="sudo transactional-update shell --continue"
alias stdc="sudo transactional-update dup --continue"
alias std="sudo transactional-update dup"
alias sfu="sudo flatpak update -y && flatpak update -y"
alias fsu="sudo flatpak update -y && flatpak update -y && sudo snap refresh"
alias dfs="pkcon update && sudo flatpak update -y && flatpak update -y && sudo snap refresh"
```
Then for example typing `std` in a terminal prompt will issue the command `sudo transactional-update dup`.

## Known Issues
### No graphical session/login screen in Hyper-V
_Hyper-V_ requires the package `xf86-video-fbdev` for graphical session:
```
$ pkcon install xf86-video-fbdev
$ sudo reboot
```

### Toolbox is not starting ("potentially insufficient UIDs and GIDs")
UUI and GUI need to be set for `toolbox` to work:
```
$ sudo tukit execute bash
  # echo "<yourusername>:100000:65536" > /etc/subuid 
  # echo "<yourusername>:100000:65536" > /etc/subgid
  # exit
$ sudo reboot
```

### Graphical applications in the toolbox are missing fonts/icons
They need to be installed inside the toolbox:
```
toolbox enter
sudo zypper ref && sudo zypper dup
sudo zypper install adwaita-icon-theme xorg-x11-fonts libX11-xcb1 gsettings-desktop-schemas
sudo gdk-pixbuf-query-loaders-64 --update-cache
```

## Alternative for Gnome Extensions
There is an alternative to installing GNOME Extensions, such that no RPM installed browser is necessary. It's currently under development at [GitHub](https://github.com/ekistece/GetExtensions). You can install it via `toolbox`; the commands are as follows:
```
$ toolbox -u
$ sudo zypper in git python38-pip
$ cd /tmp
$ git clone https://github.com/ekistece/GetExtensions.git
$ pip3 install ./GetExtensions --user
$ exit
```
You can then open the application _Get Extensions_ that is available from your launchpad.

To uninstall:
```pip3 uninstall getextensions```
