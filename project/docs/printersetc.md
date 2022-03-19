## Printing
openSUSE distributions install with fairly restrictive firewall policies. The policies may prevent printers discovery, and disallow you from successfully printing even with a correctly configured _CUPS_ printer (for example installed using the `YAST Printer` module).

In that case the software responsible for securing your computer against malicious traffic is likely `firewalld`. There's [an extensive article](https://en.opensuse.org/SDB:CUPS_and_SANE_Firewall_settings) on the _openSUSE_ wiki but here's the easiest way to check if this is your underlying issue:

1. Check if `firewalld` is indeed running via `systemctl status firewalld.service`.
2. Presuming it is, leave `YaST Printer` and temporarily suspend `firewalld` with `sudo systemctl stop firewalld.service`.
3. Go back to `YaST Printer` and try again to detect printers. If the printer(s) you're looking for now show up, then that was your issue.
4. Complete the configuration (more details on that below), and perform a test print.
5. Turn your firewall back on, for the security of your system. You can do that with `sudo systemctl start firewalld.service`.
6. Do another test print. If it works, great. If not, to get printing working again with the firewall in place, see the section on "printing through the firewall," below.

## Configuring your printer

Getting your (likely networked) printer to show up in the detected printers box in `YaST Printer` is likely not the whole story. Here are some issues and recommendations:

1. The same printer may show up on multiple lines, with different "URI"s corresponding to different protocols for accessing it. Here's how to choose:
    * If one of the URIs starts with `ipp:` -- for "internet printing protocol," more or less the "native tongue" of the printing system _CUPS_ that _openSUSE_ uses, try that first.
    * If not, then you may need to go through your options by trial and error, selecting one, configuring it, and seeing if the test print works.
    * A good second choice to an `ipp:` URI is one that begins with `socket:`.
2. You need to select a driver for the printer. If the system has been able to detect a manufacturer and/or model number from the printer, it will have filled in the driver search box below the list of connections. If not, you can type or refine this information yourself.
    * Hopefully a driver comes up that appears to be an exact or near-exact match to your printer. If not, your first option is the __Driver Packages__ button. It allows you to install the packages available for your distribution that have a variety of printer drivers.
    * If after installing the packages that seem relevant to your printer, you still don't seem to have an adequate match, try a web search for your exact model number together with "linux driver." Many manufacturers do provide installers or _CUPS_ printer definition files that will work.
    * If you're still unable to find a matching driver, you can use the __Find more__ button just to the left of "Driver Packages." That will bring up a list of very generic printer definition files. Selecting one that seems to broadly describe your printer may allow you to perform basic prints, even if you may not be using your printer's most advanced capabilities.
3. When you have selected a satisfactory driver, choose the default paper size you load into your printer, set or revise the name that the printer will go by in print dialogs, and hit OK. This will create the printer definition, and take you back to the list of configured printers, where you can perform a test print.

Be aware that this process may require trial and error, so be prepared to remove the configuration and try again with a different connection and/or driver if it doesn't work the first (or second...) time.

## Printing through the firewall

Suppose you have configured your printer and successfully printed with the firewall disabled, but are no longer able to print with the firewall turned back on. The solution should not be to leave the firewall disabled; especially when you are connected to a public network, the firewall provides important security against potential attacks.

To maintain better security, tailor the firewall to your specific situation. Presuming you are running `firewalld`, that's easiest with "firewall-config," as follows.

1. You may need to install this package with `sudo zypper install firewall-config`.
2. After starting and authenticating firewall-config, you should see the active internet connection(s) in a panel on the left. Note that each connection is assigned to a "zone" -- basically a named collection of firewall policies. If this is your first time using firewall-config, all connections are assigned to the "public" zone.
    * You should not modify the public zone; that should remain your most secure zone for using an unfamiliar or untrusted internet connection. Instead, you should take your known, trusted connections and assign them to a different zone. For example, you might take a connection to your home Wi-Fi and assign that to the (likely pre-existing) zone called "home." (You can do this by clicking on the connection in the list and then the __Change Zone__ button.)
    * You can authorize more services in the "home" zone, so that when your computer connects in a new way (such as the coffee-shop Wi-Fi), that new connection will be assigned to the default, highly secure, "public" zone.
3. To get printing working in "home", select it from the list of zones in the "Configuration" panel, and select the "Services" tab to its right. Then check the (ideally minimal set of) necessary services on the right; likely candidates for network printing are "mdns", "snmp", "ipp", and/or "ipp-client". Once a test page is again printing properly, you can simply close firewall-config; it automatically permanently saves its settings.

## Other devices

Similar considerations apply to other devices on your network that you might want your computer to interact with: scanners, plotters, 3D printers, home automation controllers, etc.

In the case of scanners, they use a protocol called SANE which is the analogue of _CUPS_, and the [detailed page](https://en.opensuse.org/SDB:CUPS_and_SANE_Firewall_settings) mentioned above has some more specific advice for SANE.

In any case, similar principles should apply: you may want to disable the firewall if configuration is not working, re-enable it when the device is responding properly, and then tailor the firewall to allow the proper operation of the device thereafter. 
