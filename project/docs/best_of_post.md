# Our "post installation" best of

Some of the material covered below is presented and discussed in more details elsewhere in this documentation. Use the links if you'd like to learn more.

The order in which you carry out these recommendations is not crucial, but we advise you to follow our order, as certain steps feel more natural if you have followed previous ones on the list. 

The list takes the point of view of a user who has just rebooted and logged in to an openSUSE distribution. Also, if you are Tumbleweed users, checkout out our [safety and usability](safety_usability.md) rules of thumbs.


### Update
Even if you have installed Tumbleweed with repositories enabled, who knows if there aren't a bunch of updates pending already? 
```
$   sudo zypper dup
```


### Take advantage of zypper's concurrent downloader
The backend for the `zypper` package manager recently got a more efficient downloader, able to download data concurrently. You can turn it on by setting the `ZYPP_MEDIANETWORK` environment variable to `1`:

```
$   export ZYPP_MEDIANETWORK=1
```

To set this variable at every boot, add the same command line to your shell's initialization file (i.e. `~/.bashrc`, `~/bash_profile`, etc).

### Get used to adding repositories with auto-refresh on
When you add repositories to Tumbleweed, those do not have their auto-refresh setting on by default. This means that doing `sudo zypper dup` might report no pending updates when in fact there __are__ updates, but those were just not seen by the package manager. To change this behavior and ensure that your system will always hook up to repositories with auto-refresh enabled, make sure you add repositories with auto-refresh on:
```
sudo zypper ar --refresh <repository url | repository file>   # you can use '-f' instead of '--refresh', they are synonyms
```

### Shell and Terminal
Even though Tumbleweed is a very accessible Linux distribution, you will have to spend some time talking to your system via a shell so better get one you feel comfortable with. We personally recommend [fish](https://fishshell.com/) and [zsh](https://ohmyz.sh/), both of which provide auto-completion for commands you have previously used and commands that are provided by the programs that the shell has discovered (including file paths!). _fish_ is a bit more friendly to the beginner, whereas _zsh_ is more feature-ful and implements a protocol that is a bit closer to the one use by the built-in shell (_bash_).

Refer to the documentation provided by the developers to make your favorite shell the default. Also, don't overlook _aliases_ -- user-defined synonyms for shell commands. You won't regret creating a few aliases should you need a command that you don't remember from a _tty_ console.

For a terminal application we recommend to stick to the one provided by your desktop environment. Users interested in having a collapsible, HUD-like terminal can turn to the excellent _Yakuake_ terminal, based on KDE's _Konsole_.

### (KDE Plasma only) Disable Discover notifications
On Tumbleweed software are better installed using the official package manager, at your own pace (although we recommend updating at least once a month) using either a terminal or YaST. This means that Discover update notifications will feel annoying at best, and will push to frantically update even in cases where you'd want to postpone updating for a couple of days or weeks. To regain your peace of mind:

Go to _Plasma settings_ > _Notification_ > _Applications: Configure_ > Select _Discover_ in the list > Untick as many boxes as you need.

### Setup your Tumbleweed for Flatpaks
Flatpaks are covered in more details in this guide, but they are an invaluable source of stable and frequently updated software:
```
$   sudo zypper in flatpak
$   flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

### Customize your .desktop files for a better user experience
Most user applications shipping their own GUI install with a desktop file (`<program name>.desktop`) providing the application with some parameters discovered at the installation. Some of these parameters are key to a comfortable user experience.

Consider for example Electron apps, which as of today (26th of July 2021) offload rendering to X11 instead of the Wayland client. This means that your Desktop Environment, which is likely to run with Wayland enabled by default, will not be able to make the most of the application's GUI.

We can improve the situation as follows:

1. Locate the application's desktop file, most likely at `/usr/share/applications`. For instance you can go a simple:
```
$   ls /usr/share/applications | grep <program name>.desktop
```
2. Once you're certain of the file name, copy it to `~/.local/share/applications`:
```
$   cp /usr/share/applications/<program name>.desktop ~/.local/share/applications
```

There you can edit the program's `.desktop` file at leisure as it won't be overwritten when updating the target program (this contrasts with editing `.desktop` files directly under `/usr/share/applications`. For instance if you wanted VS Code -- an Electron application -- to use Wayland, you could change the line:
```
Exec=/usr/share/code/code --unity-launch %F
```
to
```
Exec=/usr/share/code/code --enable-features=UseOzonePlatform --ozone-platform=wayland 
```
It might take a reboot for your applications to register changes to their `.desktop` file.

### Codecs
openSUSE distributions do not ship with proprietary software, and for most cases there are Free and Open Source alternatives, such as the free codecs shipped with certain applications (most modern web browsers such as the Chromium-based ones and Firefox).

If however you do need proprietary codecs, you can add them manually. To do this we recommend using the `opi` -- Open Build Service Package Installer, which is able to fetch software sitting just outside of the official system repositories:

```
sudo zypper in opi
opi codecs
```

Notice that some applications come packaged with proprietary codecs; that is the case for `VLC` for instance. If you install these applications from the official repositories you don't need to install codecs manually.

### Set up your GPU driver
Check out the following documentation for this step:

* [single GPUs](install_proprietary.md)
* [integrated + dedicated GPUs](hybrid_graphics.md)

### Use a password manager
KDE's _kwallet_ and _gnome-keyring_ are decent password managers as far as storing identifiers meant to be consumed by third-party applications, but some users might feel they fall short when it comes at providing a comfortable interface for auto-completion and daily management (especially the latter). 

For alternatives you can consider [KeePass Password Safe](https://keepass.info/) and [Password Safe](https://gitlab.gnome.org/World/PasswordSafe).

### Have your backup plan ready
Tumbleweed snapshots protect your system by allowing you to rollback to and from the snapshot of your liking. One noticeable exceptions to this protection is `/home` (for the simple reason that it allows the user to backup their `home` at their own pace, independently from system snapshots.)

To protect your users' `/home` directory you may want to use a third-party application to manage user-defined backups. We can recommend [restic](https://restic.net/) and the easier to the new user [rclone](https://rclone.org/).

!!! info
    If `/boot` is not on the same partition as `/`, it won't be covered by system snapshots either. In that case your bootloader (e.g. _GRUB)_ will not benefit from snapshots.

### Fine-tune Btrfs snapshots settings
Tumbleweed comes pre-configured with a generous _Btrfs_ snapshot policy that is likely to produce more snapshots than you need, leaving too significant a footprint on your storage device to be really worth it. If that is the case, please refer to the documentation on [Snapper](snapper.md) to change the defaults. Bear in mind that this is a fundamentally important aspect of Tumbleweed, so read the documentation thoroughly.
