## Pairing a Bluetooth device across different operating systems

### Shared chipset for Wi-Fi and bluetooth functionalities
Some machines -- especially laptops -- feature a single chip handling both wireless and bluetooth connections. Certain Linux drivers have a hard time communicating with microcontrollers using this architecture. This means that you may run into issues when trying to pair up or connect a bluetooth device. If this is the case:

1. Turn off all network connections (enable "Fly mode", represented with an airplane icon in most Linux desktop environments.)
2. Enable bluetooth, and pair up or connect the device.
3. Turn on network connections (disable "Fly mode").

This should allow you to have both functional and bluetooth connections.

### Pairing up the same device across different operating systems on the same machine
When you pair up a bluetooth device to your machine for the first time, they negotiate a single identifier key that will be used to authenticate the device whenever you actually connect it. An important caveat is that there is no built-in mechanism to allow you to connect the bluetooth device to an operating system (on the same machine) that is different from the operating system where the pairing up took place. This is unpleasant if you happen to use several operating systems on the machine (multi-boot scenario).

You can however circumvent this issue by manually re-applying the same key across operating systems. The procedure below has been tested across any operating system (except macOS). It works across different Linux distributions (you will just need steps 10-14 then).

__Windows 10 and Linux__ (adapted from [StackExchange](https://unix.stackexchange.com/questions/255509/bluetooth-pairing-on-dual-boot-of-windows-linux-mint-ubuntu-stop-having-to-p))

1. Pair all devices on Linux 
2. Pair all devices on Windows
3. From Windows, go to _Device & Printers_ in _Control Panel_ and go to your Bluetooth device's properties. Then, in the _Bluetooth_ section, you can find the unique identifier. Copy it as you will need it.
4. Download __PsExec__ from [here](http://technet.microsoft.com/en-us/sysinternals/bb897553.aspx).
5. Unzip the zip you downloaded and open a _CMD_ window with elevated privileges. (Click the Start menu, search for cmd, then right-click the CMD and click __Run as Administrator__.)
6. `cd` into the folder where you unzipped your download.
7. Run `psexec -s -i regedit.exe`
8. Navigate to find the keys under `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\BTHPORT\Parameters\Keys`.  If there is no _CurrentControlSet_, try _ControlSet001_.
9. You should see a few keys labels with the MAC addresses -- write down the MAC address associated with the unique identifier you copied before.
10. Go back to Linux (if not in Linux) and add your Windows key to your Linux config entries. Just note that the Bluetooth port's MAC address is formatted differently when moving from Windows to Linux - referenced as aa1122334455 in Windows in my example above. The Linux version will be in all caps and punctuated by ':' after every two characters - for example `AA:11:22:33:44:55`.  
11. Switch to root: do `sudo -s`
12. `cd` to your Bluetooth config location at `/var/lib/bluetooth/[bth port  MAC addresses]`
13. Here you'll find folders for each device you've paired with. The folder names being the Bluetooth devices' MAC addresses and contain a single file info. In these files, you'll see the link key you need to replace with your Windows ones, like so:
```
[LinkKey]
Key=< NEW KEY, ex. B99999999FFFFFFFFF999999999FFFFF >
```
14. Finally restart your bluetooth systemd service with `sudo systemctl restart bluetooth`
