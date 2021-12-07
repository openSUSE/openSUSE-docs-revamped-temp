## Installing Codecs 
You need to play online or offline multimedia content, but the content does not play or you receive errors. Usually this is a sign of missing codecs. Due to legal limitations proprietary codecs can't be stored and served directly from the __openSUSE__/__SUSE__ infrastructure. To install proprietary codec packages you will need to add the [Packman](http://packman.links2linux.org/) repository and install the required software from there. Some commonly used and installed codecs are:

- ffmpeg
- gstreamer-plugins-good
- gstreamer-plugins-bad
- gstreamer-plugins-libav
- gstreamer-plugins-ugly
- libavcodec-full
- vlc-codecs

!!! info
    Tumbleweed users who only occasionally use codecs (for example through a web browser or a handful of dedicated applications such as _VLC_ or _OBS Studio_) might find it convenient _to avoid_ the addition of external repositories by using a version of these applications shipping with their own codecs. On Tumbleweed the advantage of this approach is more salient as codecs will be kept separate from system libraries, securing extra stability for the user. Interested readers may refer to [this page](alternative_procurement.md#flatpaks) for explanations.

You could get codecs using `Zypper` or `YaST` but you _should_ get them with _OBS Package Installer_ (`opi`) as there are complications and risks associated with the alternative methods.

## Installing the Open Build Service Package Installer (opi) 
`opi` can be used to search and install software from the _Open Build Service_ (OBS) and it works on _Leap_ and _Tumbleweed_. The only limitation is that this method will consider Packman as the unique provider of all codecs — not just the proprietary ones which openSUSE cannot provide. Proceed if this limitation is acceptable for you. If it is not acceptable, consider using `flatpaks` (see the __Info__ above).

__From a terminal__:

1. Install `opi`, the _Open Build Service_ command line utility: `sudo zypper install opi`
2. Run the installer for Packman-provided codecs: `opi codecs`
3. Confirm the prompt (`y` — if your system language is English — and then `ENTER`).

As of this writing, `opi` performs the following operations behind the curtains (see [here](https://github.com/openSUSE/opi/blob/d880d81fb315838e17051ee518477498ee5ffd96/opi/plugins/packman.py#L15) and [there](https://github.com/openSUSE/opi/blob/d880d81fb315838e17051ee518477498ee5ffd96/opi/__init__.py#L62) for reference):

1. Add the Packman repository with a higher priority than the official repositories.
   + `sudo zypper ar -cfp 90 'https://ftp.gwdg.de/pub/linux/misc/packman/suse/openSUSE_Tumbleweed/' packman`
2. Upgrade the system exclusively using the Packman repository.
   + `sudo zypper dist-upgrade --from packman --allow-downgrade --allow-vendor-change`
3. Install the codecs to the system.
   + `sudo zypper in --from packman ffmpeg gstreamer-plugins-bad gstreamer-plugins-libav gstreamer-plugins-ugly libavcodec-full vlc-codecs`
   + `sudo zypper in gstreamer-plugins-good gstreamer-plugins-good-extra`


Upgrading and installing packages using the option `--from packman` tells the system to set Packman as their unique, permitted provider. This means that updates from the openSUSE official repositories targeting the same packages will _no longer_ be applied. This behaviour is intended for users who agree to have all their codecs provisioned by Packman.
