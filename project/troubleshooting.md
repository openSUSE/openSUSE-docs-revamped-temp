## Troubleshooting

### Introduction
Being able to troubleshoot your issues yourself is an important factor for a rewarding experience of Linux. If you have no autonomy in troubleshooting your own issues, you won't be able to help others help you, as you won't know how to relevantly describe your issue, let alone share meaningful and relevant data. No autonomy also means that you'll likely miss one beautiful aspect of the Linux culture, which is to build an understanding of how your system works, tinker and help others -- this all starts with some know-how about troubleshooting.

A schematic representation of your system goes like this:

```
Applications (i.e. Firefox)
Desktop Environment (i.e. GNOME, KDE Plasma)
System libraries (i.e. glibc)
System and service manager (i.e. systemd)
Linux kernel
```

Roughly speaking every "layer" in the above diagram depends on the ones beneath it. This means that failures in one layer might affect layers above but not under it. This basic fact will help you overcome the first challenge to troubleshooting issues on Linux, namely _identifying the layer of the relevant cause_.

!!! info
    Generally speaking you can you `man <...>` to get the manual pages pertaining to any command associated with a manual. For example, to get the manual page about __journalctl__ (described below), simply run `man journalctl`. And if you want to dig deeper to a specific option, for instance the `-p` option, you can do `man journalctl -p`.

### Journalctl
`journalctl` is a systemd unit that will often be your best (and unique!) friend in troubleshooting issues. The three most frequent scenarios where you'd need to use it:

_Crashes, freezes and other machine-halting issues_. In that case you'll typically want to reboot and display the logs from the previous boot. For this you'll need a command of the type `journalctl -b -1 <additional options, arguments and flags>`. `-b -1` jointly mean: "Select the logs from the previous boot."

_Issues since the start of the current boot_. Perhaps you're having issues since booting your machine and you are interesting in displaying only the logs registered from the current boot. For this you'll need a command of the type `journalctl -b`. `b` means: "Select the logs logged since the current boot". 

_Narrowing down on a specific application or services_. Suppose you have found a "suspect" and you want to test your hypothesis that it might be involved in the issue under consideration. It comes in handy to display the logs in real-time, as you test out the suspicious program. Then you'll need a commend of the type: `journalctl --follow`. `--follow` means: "Display all logs in real-time from now on".

Linux logging mechanisms are quite verbose and you'll probably swim in data which are not connected to the issue you're trying to troubleshoot. One useful otion for filtering out irrelevant data is the `-p <n...m>` option, as in:

```
$   journalctl -b -p0..4
```

which selects all the logs registered since the current boot while filtering out logs whose value of import is not in the 0-4 interval. There are 8 values, determined by the `syslog` specificiation: emergency (0), alert (1), critical (2), error (3), warning (4), notice (5), info (6), and debug (7). So `0..4` means to keep only emergency, alert, critical, error and warning logs.

If you read this document chances are that the cause of your issue(s) is located within the 0-4 interval, and most likely within the 0-3 interval. You'll need to play around the 3/4 bound and find for yourself.

### Systemctl
`systemctl` is __systemd__'s command-line utility for observing and managing all things related to systemd services. As some applications rely on specific systemd services to run, inspecting these will come in handy.

To display all systemd units -- the various components implementing systemd services -- you can do:

```
$   systemctl list-units -t target --all

```

To display the status of a specific unit, e.g. `bluetooth`:

```
$   systemctl status bluetooth
```

To display the properties a specific unit:

```
$   systemctl show bluetooth
```

The former are mostly determined by the configuration file of the appropriate systemd unit file, which are concrete files on your system you can inspect using ordinary tools. Most of the configuration files -- as well a recent log pertaining to the service -- can be found using the `status` command.

### Systemd-analyze ("fixing slow boots")
__systemd__ comes with tools to inspect its own initialization sequence. `systemd-analyze` come in handly for fixing some cases of a "slow boot".

```
$   systemd-analyze critical-chain
```

A typical output:

```
graphical.target @9.161s
└─multi-user.target @9.161s
  └─plymouth-quit-wait.service @1.752s +4.417s
    └─systemd-user-sessions.service @1.744s +6ms
      └─network.target @1.741s
        └─networking.service @1.610s +131ms
          └─apparmor.service @1.359s +149ms
            └─local-fs.target @1.358s
              └─boot-efi.mount @1.236s +122ms
                └─dev-sda1.device @1.225s
```

!!! info
    You can output an SVG-rendered version of the data with `systemd-analyze plot`. For writing the result to a file in your home directory: `systemd-analyze plot > boot_analyze_graph.svg`.

The story that this tells is that it took about 9 seconds for my system to reach land me onto my login manager ("graphical target"), with about 4 full seconds spent on the `plymouth-quit_wait.service`.

A useful command to pair up with `systemd-analyze critical chain` is `systemctl list-dependencies`. This command allows you to walk the list of services that depend on on a target service, or vice-versa, services on which the target service depends. Let's try:

```
$   systemctl list-dependencies --reverse plymouth-quit-wait.service
plymouth-quit-wait.service
● └─multi-user.target
●   └─graphical.target
```

These are the services which depend on `plymouth-quit-wait`. And now the services on which `plymouth-quit-wait` depends on:

```
systemctl list-dependencies plymouth-quit-wait.service
plymouth-quit-wait.service
● ├─system.slice
● └─sysinit.target
●   ├─apparmor.service
●   ├─blk-availability.service
●   ├─dev-hugepages.mount
●   ├─dev-mqueue.mount
●   ├─finalrd.service
●   ├─keyboard-setup.service
●   ├─kmod-static-nodes.service
●   ├─lvm2-lvmpolld.socket
●   ├─lvm2-monitor.service
...etc.
```

The uptake is that very little depended on `plymouth-quit-wait` while the service depended on a lot of things. This falls short of a demonstrative proof, but it suggests that the service's initialization was likely not the cause of a "slow down". The culprit -- if any -- should be searched for between the many services on which it depended was waiting for.

If on the other hand you find a genuine culprit in systemd initialization sequence, you can mask it:

```
$   systemctl stop <service name>
$   systemctl mask <service name>
```

Masking a service means that it won't be automatically loaded during your next boot, and that it won't be loaded either if a service that depends on it is loaded. You can reverse the masking with:

```
$   systemctl unmask <service name>
```

and start using it now with `systemctl restart <service name>`.