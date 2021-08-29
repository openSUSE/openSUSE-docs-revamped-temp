## Printing

Getting hard copy out of your __openSUSE__ box should be pretty straightforward with a directly-connected (via USB, say) printer. It will likely show up in print dialogs with no action on your part and "Just Work." If not, or typically if you want to print to a networked printer, then you will have to set the printer up in advance. The most straightforward alternative is the `YaST Printer` module. When you start it up, you will see a list of your existing printer configurations, which will initially be empty.

!!! warning
    Should there be a screenshot here? Or are we not going into that level of detail/handholding?

To configure a new printer, click "Add" and `YaST` will do an initial scan for printers, as well as for printer drivers. Often this initial scan comes up short, so just go ahead and click on the "Detect More.." button in the upper right. At this point, you should generally see a list of all of the printers that are visible on your current local networks (and that are powered on and responding to network traffic, and so on). Note that some printers may show up on multiple lines, with different "URI"s corresponding to different protocols for accessing that printer.

However, suppose you have a printer on your local network that you are pretty certain should be accessible -- for example, that's especially the case if you've been able to identify its IP address on the local network, and perhaps access it through a browser interface, or verified that it responds to a `ping <IP-ADDR>` command. But perhaps that printer steadfastly refuses to show up on the list of detected printers, even with the "Detect More..." operation. In that case, a likely suspect is the system's software firewall.

If you performed a relatively "vanilla" installation of a recent version of Tumbleweed, for example, mostly taking default options, chances are that a firewall was set up and configured with fairly restrictive policies -- for your security in potentially unknown and risky environments. Those default policies quite possibly are sufficiently stringent to prevent printer discovery, and even to prevent successfully printing with a correctly configured "CUPS" printer.

Again, on recent installations, the software responsible for securing your computer against malicious traffic is likely "firewalld." There's [an extensive article](https://en.opensuse.org/SDB:CUPS_and_SANE_Firewall_settings) on the _openSUSE_ wiki about the nuances of network printing and firewalls, but the here's the easiest way to check if this is your underlying issue.

First, check if firewalld is indeed running via `systemctl status firewalld.service`. Presuming it is, leave `YaST Printer`, temporarily suspend firewalld with `sudo systemctl stop firewalld.service`, and then go back to `YaST Printer` and try again to detect printers. If the printer(s) you're looking for now show up, then indeed that was your issue. Go ahead and complete the configuration (more details on that below), and perform a test print.

Once you've got that working, you'll want to turn your firewall back on -- after all, it is important for the security of your system. You can do that with `sudo systemctl start firewalld.service`. But now be sure to do another test print. If it works, great. However, it may well not -- the policies in place can be sufficiently strict that they actually prevent printing to a correctly configured network printer. To get printing working again with the firewall in place, see the section on printing through the firewall, below.

## Configuring your printer

Getting your (likely networked) printer to show up in the detected printers box in `YaST Printer` is the biggest part of the battle, but it's not necessarily the whole story. First, if the same printer has shown up on multiple lines, which one should you choose? If one of the URIs starts with `ipp:` -- for "internet printing protocol," more or less the "native tongue" of the printing system CUPS that _openSUSE_ uses, try that first. If not, then you may need to go through your options by trial and error, selecting one, configuring it, and seeing if the test print works. A good second choice to an `ipp:` URI is one that begins with `socket:`.

Second, once you've chosen a connection to the printer, in most cases you need to select a driver for it as well. If the system has been able to detect a manufacturer and/or model number from the printer, it will have filled in some search terms in the driver search box below the list of connections. If not, you can type this information yourself.

Hopefully a driver comes up that appears to be an exact or near-exact match to your printer. If not, your first option is the "Driver Packages" button. With that, you can easily install the packages available for your distribution that have a variety of printer drivers.

If after installing the packages that seem relevant to your printer you still don't seem to have an adequate match, definitely try an internet search for your exact model number together with "linux driver." Many manufacturers do provide installers or CUPS printer definition files that will work.

If you're still unable to find a matching driver, you can use the "Find more" button just to the left of "Driver Packages." That will bring up a list of very generic printer definition files. Selecting one that seems likely to broadly describe your printer may well allow you to perform basic prints, even if you may not be using your printer's most advanced capabilities.

When you have selected a satisfactory driver, choose the default paper size you load into your printer, set or revise the name that the printer will go by in print dialogs, and hit OK. This will create the printer definition, and take you back to the list of configured printers, where you have the option of performing a test print. If you're in doubt about the configuration, that's generally a good idea. And be aware that this process can require some amount of trial and error, so be prepared to remove the configuration and try again with a different connection and/or driver if it doesn't work the first (or second...) time.

## Printing through the firewall

Suppose you have configured your printer and successfully printed with the firewall disabled, but are no longer able to print with the firewall turned back on. The solution should not be to leave the firewall disabled; especially when you are connected to a public network, like at a coffee shop, or directly connected to the internet (not through a router that may be providing its own firewall services), the firewall provides important security against potential attacks.

To maintain better security, the answer is to tailor the firewall to your specific situation. Presuming you are running firewalld, that's easiest with "firewall-config," which you may need to install with `sudo zypper install firewall-config`.

When you start up firewall-config, it will first need to authenticate to allow priviliges to inspect and modify the firewall. Then you should see the active internet connection(s) in a panel on the left. Note that each connection is assigned to a "zone." A zone basically just means a named collection of firewall policies. If this is your first time using firewall-config, likely all connections are assigned to the "public" zone, which is the initial default.

You should not modify the public zone; that should remain your most stringently secure zone that you can fall back to whenever you are using an unfamiliar or untrusted internet connection. Instead, you should take your known, trusted connections and assign them to a different zone. For example, you might take a connection to your home wifi, which is perhaps generated by your broadband router and behind a firewall that router provides, and assign that to the (likely pre-existing) zone called "home." (You can do this by clicking on the connection in the list and then the "Change Zone" button.)

Now you can authorize more services in the "home" zone, secure in the knowledge that when your computer connects in a new way (such as the wifi service at your library), that new connection will be assigned to the default, highly secure, public zone.

To get printing working in whatever less stringent zone you choose to set up, select it from the list of zones in the "Configuration" panel, and select the "Services" tab to its right. Then check the (ideally minimal set of) necessary services on the right; likely candidates for network printing are "mdns", "snmp", "ipp", and/or "ipp-client". Once a test page is again printing properly, you can simply close firewall-config; it automatically permanently saves its settings.

## Other devices

Similar considerations apply to other devices on your network that you might want your computer to interact with: scanners, plotters, 3D printers, home automation controllers, etc. In the particular case of scanners, they use a protocol called SANE which is the analogue of CUPS, and the [detailed page](https://en.opensuse.org/SDB:CUPS_and_SANE_Firewall_settings) mentioned above has some more specific advice for SANE. In any case, similar principles should apply: you may want to disable the firewall if configuration is not working, re-enable it when the device is responding properly, and then tailor the firewall to allow the proper operation of the device thereafter. 
