# KDE Connect

__KDE Connect__ is a project that enables all your devices to communicate with each other. Here's a few things KDE Connect can do:

* Receive your phone notifications on your desktop computer and reply to messages
* Control music playing on your desktop from your phone
* Use your phone as a remote control for your desktop
* Run predefined commands on your PC from connected devices. See the list of example commands for more details.
* Check your phone's battery level from the desktop
* Ring your phone to help find it
* Share files and links between devices
* Browse your phone from your desktop
* Control the desktop's volume using your phone
* Send SMS from your desktop

## Installation
KDE Connect is available for openSUSE Leap and Tumbleweed from the main repository. Use the following instructions, should you need to install it.

!!! info 
    If you use a desktop environment other than KDE's Plasma, you might also want to install `indicator-kdeconnect`, that provides a system tray as a GUI for other desktops.

### Using YaST

![YaST Software Management](image/yast_kdeconnect.png)

1. Launch `YaST`
1. Select `Software Management`
1. Type `kdeconnect` in the `Search` field and click __Search__
1. In the result list select `kdeconnect-kde`
1. Click __Acept__ to confirm the installation

### Using the terminal
1. Refresh your local repository `sudo zypper ref`
1. Run `sudo zypper install kdeconnect-kde`

### Android Installation
The app for Android can be found in both the [Google Play Store](https://play.google.com/store/apps/details?id=org.kde.kdeconnect_tp) and the free store [F-Droid](https://f-droid.org/repository/browse/?fdid=org.kde.kdeconnect_tp).


### Firefox Extension
If you are a Firefox user, this [extension](https://addons.mozilla.org/en-GB/firefox/addon/kde_connect/) might be of your interest.

## Configuration
In order for the secure communication to take place, ports in the firewall have to be opened up. We can do this using _YaST Firewall_ or using the command _firewall-cmd_ from the terminal.

!!! info
    Note the zone of your current interface. If you do not specify the zone of the interface, it defaults to public.

### Using YaST Firewall to open the ports
![YaST Firewall](image/yast_firewall.png)

1. Launch `YaST`
1. Select `Firewall`
1. Within the `Zones` list, select `Public`
1. From the list of `Known` services search for `kdeconnect`, then click __Add__
1. Click __Accept__
1. The Firewall Service settings will save and the service will restart

### Using firewalld-cmd from the terminal to open the ports
The easiest way to configure firewalld is through the terminal.

The example below is adjusting the default zone using the kdeconnect service file for firewalld (`/usr/lib/firewalld/services/kdeconnect-kde.xml`). This service holds the list of ports required for a proper operation. For most user cases, this is an acceptable default.

```
$ sudo firewall-cmd --zone=public --permanent --add-service=kdeconnect-kde
$ sudo firewall-cmd --reload
```

## Android Configuration
Most networks should allow automatic discovery of the devices running KDE Connect.

### Pairing devices
1. Launch `KDE Connect` on your computer
1. Launch `KDE Connect` application on the Android device
1. In the Android device, within `Available devices` select your computer
1. Tap `Request pair`

Once paired, scroll through the options to determine how much information you want communicated back and forth between devices.

## Troubleshooting
### Devices are running KDE Connect, but they can't see each other
* Ensure that the devices in question are on the same network
* Ensure the necessary firewall ports have been opened. If they have been opened, try restarting the firewall daemon using the YaST Firewall Tool
Go to `KDE Connect Settings` (on Android and/or openSUSE) and manually refresh the devices
### KDE Connect crashes or restarts when trying to pair with another device
* Some times, a corrupt config file may cause KDE Connect to crash when trying to pair with a device. In that case, deleting the config files in `~/.config/kdeconnect` might help.
### The devices suddenly stop communicating
If both devices are on the network and the firewall is properly configured but no notifications are being displayed one or the other may device have fallen asleep. Likely the service stopped or was suspended on the mobile device.

Open the KDE Connect settings on either device and ping the opposite or select Refresh to force connection.


## Learning more
* [KDE Connect site](https://kdeconnect.kde.org)

## References
* [SDB:KDE Connect](https://en.opensuse.org/SDB:KDE_Connect)
    * [authors](https://en.opensuse.org/index.php?title=SDB:KDE_Connect&action=history)
* [KDEConnect](https://community.kde.org/KDEConnect)