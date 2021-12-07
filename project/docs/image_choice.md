## Choosing an installation media
The installers and images can be obtained on the [Tumbleweed](https://software.opensuse.org/distributions/tumbleweed) and [Leap](https://software.opensuse.org/distributions/leap) pages.

Most users are used to Live images, packing a full-blown environment where they can test
out a Linux distribution before installing.

!!! note for Tumbleweed
    Users interested in Tumbleweed should bear in mind that, because of the way Tumbleweed is built, Tumbleweed Live images are missing a handful of tests in comparison to Tumbleweed DVD images (see below). So even if Live images can be considered reasonably safe for most users, we cannot guarantee the same quality assurance as for DVD images.

Our live images (~950 MBs) provide the following desktop environments:

- KDE
- GNOME
- XFCE

They each come in different flavours tailor-suited to your CPU architecture:

- x86_64 (64-bit)
- i686 (32-bit)
- aarch64 (Armv8-A "ARM" 64-bit)

The two other installation media we provide are DVD images and Network installer.

- DVD images are quite bulky (~4.5 GBs), but allow you to install an entire distribution without relying on network for retrieving components.
- Networks installers are much lighter (~150 MBs); they require a stable and hopefully fast connection to fetch components over the network.

## Authenticity & integrity checks (*NIX)

!!! note
    The following two steps are optional, but are considered good practice.

You perform an _integrity check_ when you want to make sure that a given file has not been tampered with. It typically involves making sure that a file's checksum matches a certain target.

You perform an _authenticity check_ when you want to make sure that a file -- typically the file that specifies the checksum target -- was obtained from a legitimate source (i.e. openSUSE). Because performing an integrity check against an inauthentic target does not make sense, you want to first do an authenticity check, and only then do an integrity check.

### Assumptions & definitions
In what follows we make the following assumptions:

1. You have downloaded an openSUSE Live, Network or DVD image from openSUSE website (hereafter: the __Image__):
    * [Tumbleweed](https://software.opensuse.org/distributions/tumbleweed)
    * [Leap](https://software.opensuse.org/distributions/leap)
2. You have obtained the __Target Checksum__ file for your image, by clicking on the _Checksum_ next to the image.
3. You have obtained openSUSE gpg __Public Key__ by importing it via its keyid into your gpg keyring with
    ```
    $  gpg --recv-keys 0x22C07BA534178CD02EFE22AAB88B2FD43DBDC284
    ```
4. You have obtained an openSUSE gpg __Detached Signature__ file corresponding to the image you want to perform the check upon. The Tumbleweed detached signature files are held within a single directory [here](http://download.opensuse.org/tumbleweed/iso/). Regarding Leap however you need to start [there](http://download.opensuse.org/distribution/openSUSE-current/) and use the `/live` subdirectory for the live images and the rescue images (the `/iso` subdirectory does not provide any detached signature for net installers and DVD images). Once you have the address of the signature file, the command looks like this (i.e. for KDE Live):
    ```
    $  wget http://download.opensuse.org/tumbleweed/iso/openSUSE-Tumbleweed-KDE-Live-x86_64-Current.iso.sha256.asc
    ```

### Authenticity check
For Tumbleweed (all images) and Leap (only live images) the authenticity check is an instance of the command:
```
$  gpg --verify <Detached Signature.asc> <Target Checksum.iso.sha256>
```
For Leap non-live images (typically DVD images) the authenticity check uses no signature (clearsign check); it is an instance of the command:
```
$  gpg --verify <Target Checksum.iso.sha256>
```
The check passes just in case the command outputs calls it a success.

!!! warning
    If the check fails, make sure you are honoring our [assumptions](#assumptions-definitions).

### Integrity check
It is an instance of the command:
```
$  sha256sum <Image.iso>
```
The check passes just in case the output of the above equals the checksum written in plain text in the __Target Checksum__ (see [Definitions & assumptions](#assumptions-definitions)).

## Authenticity and integrity checks (Windows 10)

__Preparation__:
1. Download the filename.iso.sha256 file from the Download section of opensuse.org or download the hash files from here

* for Tumbleweed: https://download.opensuse.org/tumbleweed/iso/openSUSE-Tumbleweed-DVD-x86_64-Current.iso.sha256
* for Leap: https://download.opensuse.org/distribution/leap/15.3/iso/openSUSE-Leap-15.3-DVD-x86_64-Current.iso.sha256

Note that it does not matter where exactly you put the files as long as they are all in the same folder and keep their original names.

2. Then browse to https://www.gnupg.org/download/index.html and download and install the Windows installer for GnuPG.

!!! info
    For the purposes of this guide it does not matter whether you install the program as administrator or not, so just click yes to install, to install without administrator rights if it asks you.

3. Find the folder containing the files you downloaded in the first step, hold Shift while right-clicking it (the folder, not the files in it). Select it to open a command window.

Use whichever command line utility you've got available, but if you used the PowerShell option, type `cmd` followed by `Enter` in the command prompt.

### Integrity Check

1. Locate the folder of the downloaded ISO image
2. Open a command prompt.
3. `cd` to the folder and finally type this command, confirming the command with `Enter`:
```
CertUtil -hashfile <filename>.iso SHA256    # replace <filename> with the actual name of your ISO image
```
Example:
```
certutil -hashfile openSUSE-Tumbleweed-DVD-x86_64-Snapshot20210611-Media.iso SHA256
```

Note that if you start typing a filename you can press Tab to automatically complete it.

This will take a little while to complete and eventually present you with an alphanumeric sequence that is called a hash. If this hash is identical to the one listed in your filename.iso.sha256 then the integrity check passed. You can compare them by hand (note that some Windows text editors will not display the line breaks in that document so it may look weirdly formatted) or better use the find command.

If the same hash is not found in `<file_name>.iso.sha256`, then your downloaded ISO image did not pass the integrity check. Make sure you downloaded the correct `<file_name>.iso.sha256` and if yes, try to download the ISO image again from a different mirror server. Try on it the integrity again.

!!! info
    On older versions of Windows, the CertUtil command does not exist. Instead it is recommended to install `7zip` from https://www.7-zip.org/, then find and right-click the .iso file your downloaded in Windows Explorer and select 7-Zip's CRC SHA option to calculate the SHA 256 hash, which must then compare to the one from the filename.iso.sha256 file yourself.

!!! warning
    Never install from an .iso that failed the integrity check.

### Authenticity Check

To perform the authenticity check of the ISO image,

1. obtain the official GPG public key and SHA256 hash file from here (http://download.opensuse.org/tumbleweed/iso/) of your required openSUSE iso version.
2. Download the required "filename.iso.sha256.asc" and "filename.iso.sha256" file and save it named with "sha256sum.txt.gpg" and "sha256sum.txt" in one folder. (Note: ISO file and these two .txt file should be in one folder).
3. You can then perform the check from a command prompt.

   gpg --verify sha256sum.txt.gpg sha256sum.txt

   {Message for you Ad: you can attach a image of the result of this command line}

As long as it says Good signature from "openSUSE Tumbleweed ISO Signing Key <root@opensuse.org>" and with a fingerprint matching the one shown above and that you used to download the key, then that means your download is authentic. In case it was tampered with the message would be BAD signature from ....

You can ignore the warning that comes after that, it is expected and perfectly normal.

## Get to the installer
In this section you will learn how to get to a running installer for _Tumbleweed_ or _Leap_. This involves:

- Using a program to write an installation image -- see [Choosing an installation image above](#choosing-an-installation-media) -- to an external device, usually a USB pen drive or some other external storage medium.
- Having the media adequately partitioned to host the image, and making it bootable
- Booting on the external media and running the installer.

We will assume that you have downloaded an installation image and optionally, have performed an authenticity check on it -- see the above sections for details. The next section also assumes that you will be using a USB flash drive as the installation media. The section after that handles external hard drives.

### Prepare the installation media (USB flash drive)
The process for preparing the installation media varies depending on the operating system from which you are following these steps. Different operating systems offer different applications. We recommend using any of the following programs, which we have tested and which are known to be simple to use and reliable on their respective platforms:
- Fedora Image Writer ([Windows & Mac](https://getfedora.org/en/workstation/download/), [Linux flatpak](https://flathub.org/apps/details/org.fedoraproject.MediaWriter))

For _Fedora Image Writer_:

1. Make sure your USB flash drive has enough storage to hold the image (either ~1 GB or ~5 GB) and plug it in.
2. Run _Fedora Image Writer_. From the main menu pick __Custom image__ and then select the `.iso` image to write to the USB flash drive. Make sure that both the image and flash drive have been correctly selected, as in: ![this screenshot](image/image_choice_fiw_conf_prompt.png)
3. Finally, click on __Write to Disk__.

Once writing the image is done, you are ready to reboot from the USB drive. Leaving the USB drive plugged in, use the method appropriate to your operating system to restart your computer.

### Reboot to the device
Assuming you have successfully installed an openSUSE distribution, you should reboot according to the method relevant to your current operating system.

#### Windows

Use [these instructions](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/boot-to-uefi-mode-or-legacy-bios-mode) to reboot to your BIOS or UEFI. From there you will be able to select the USB flash drive hosting the installation image as boot destination.

#### macOS
Simply hold the `alt` (option) key immediately after restarting and select the openSUSE bootloader from there.
