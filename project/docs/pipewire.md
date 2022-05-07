### What is pipewire?

According to the [pipewire website](https://pipewire.org/)

> [It] is a project that aims to greatly improve handling of audio and video under Linux. It provides a low-latency, graph based processing engine on top of audio and video devices that can be used to support the use cases currently handled by both pulseaudio and JACK. PipeWire was designed with a powerful security model that makes interacting with audio and video devices from containerized applications easy, with supporting Flatpak applications being the primary goal. Alongside Wayland and Flatpak we expect PipeWire to provide a core building block for the future of Linux application development.

TLDR; Pipewire is a new multimedia framework that aims for minimal latency and improved security with support for Pulseaudio, JACK, ALSA, and GStreamer.

### Installing

!!! warning 
	This removes the package `pulseaudio`.

Install the package:

```
sudo zypper in pipewire pipewire-pulseaudio
```

Then enable the pipewire audio socket:

```
systemctl --user enable --now pipewire.{service,socket}
systemctl --user enable --now pipewire-pulse.{service,socket}
systemctl --user enable --now pipewire-media-session.service
```

Finally reboot your system:

```
systemctl reboot
```

## References & credits
- [openSUSE:Pipewire](https://en.opensuse.org/openSUSE:Pipewire)
    - [authors](https://en.opensuse.org/index.php?title=openSUSE:Pipewire&action=history)