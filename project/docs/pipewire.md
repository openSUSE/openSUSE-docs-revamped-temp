### What is pipewire?

### Installing

Warning: This removes the package `pulseaudio`

Install the package:

```
sudo zypper in pipewire pipewire-pulseaudio
```

Then enable the pipwire audio socket:

```
systemctl --user enable --now pipewire.{service,socket}
systemctl --user enable --now pipewire-pulse.{service,socket}
systemctl --user enable --now pipewire-media-session.service
```

Finally reboot your system:

```
systemctl reboot
```
