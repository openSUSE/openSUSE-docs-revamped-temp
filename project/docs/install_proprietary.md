## Nvidia proprietary drivers

### Determine which driver is required by your GPU

1. There are three Nvidia driver options available in the Nvidia repository - __nvidia-glG06__, __nvidia-glG05__ and __nvidia-glG04__.
2. The __nvidia-glG06__ package corresponds with the Nvidia 515 series driver.
3. The __nvidia-glG05__ package corresponds with the Nvidia 470 series driver.
4. The __nvidia-glG04__ package corresponds with the Nvidia 390 series driver.
5. Please refer to the Nvidia [website](https://www.nvidia.com/en-us/drivers/unix/) to determine which driver best supports your GPU.

### Setup the driver

#### With YaST
1. Go to _YaST2_.
2. Then _Software Management_.
3. On the menu, click __Configuration__ &gt; __Repositories__... (or do `Ctrl + R`).
4. Click __Add__ &gt; __Community Repositories__.
5. Select __nVidia Graphics Drivers__ &gt; __Accept__ &gt; __Trust__.
6. On the _Configured Software Repositories_ &gt; Click __Ok__
7. Back to _Software Management Windows_" &gt; __View__ &gt; __Repositories__ &gt; Select __nVidia Graphics Drivers__.
8. Select __x11-video-nvidiaG06__ &gt; __Accept__ (Some graphic cards need _G05_ or _G04_, see the first section above)
9. Reboot.

#### Using the command line
1. Add the Nvidia Repository. If using Tumbleweed for example, you would run `sudo zypper addrepo --refresh https://download.nvidia.com/opensuse/tumbleweed NVIDIA`. For Leap, you can run `sudo zypper addrepo --refresh 'https://download.nvidia.com/opensuse/leap/${releasever}' NVIDIA`.
2. To auto-detect and install the right driver for your hardware, run: `sudo zypper install-new-recommends --repo nvidia` Alternatively you can also specify which version via: `sudo zypper in x11-video-nvidiaG06` or `sudo zypper in x11-video-nvidiaG05` or `sudo zypper in x11-video-nvidiaG04`
3. Reboot.

#### CUDA
1. CUDA can be installed with the __Nvidia-ComputeG05__ or __Nvidia-ComputeG04__ package.
2. See [Nvidia's documentation](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html) for further information.

#### Manual installation

!!! warning 
    This setup is not supported by your distribution. Make sure you know what you are doing and you know the risks of installing NVIDIA the manual way. Before doing this, make sure you have created a snapshot so you can rollback if something goes wrong e.g. `sudo snapper create`.

Sometimes, installing NVIDIA through zypper is not what you want because you want to either get the latest driver from the new feature branch or install only the things you "needed". Thus, you will do a manual install of NVIDIA. In preparation, run `sudo systemctl isolate multi-user.target` and to disable your display manager, run `sudo systemctl disable --now display-manager.service`. Make sure Xorg or an existing X session is not running.

1. Check in `zypper` if you have `NVIDIA` repo. If there is, remove NVIDIA repo with `sudo zypper removerepo NVIDIA`.
2. Uninstall the existing NVIDIA packages with the following commands:
```sh	
$ sudo zypper remove x11-video-nvidiaG06 nvidia-glG06 # or `sudo zypper remove x11-video-nvidiaG05 nvidia-glG05` or`sudo zypper remove x11-video-nvidiaG04 nvidia-glG04` depending on the driver you are using.
$ sudo zypper ref -rf  # to refresh all repositories.
```
3. Do note that NVIDIA has two "download types"; the new feature branch and production branch. Download NVIDIA.run or `nvidia-installer` with your chosen download type through your browser or through `curl` or `wget` from either of these websites:
    - [https://www.nvidia.com/Download/Find.aspx](https://www.nvidia.com/Download/Find.aspx)
    - [https://www.nvidia.com/Download/index.aspx](https://www.nvidia.com/Download/index.aspx)
    - [https://http.download.nvidia.com/XFree86/nvidia-installer/](https://http.download.nvidia.com/XFree86/nvidia-installer/)
4. Let's say you downloaded NVIDIA into your `$HOME` from the new feature branch which as of writing is NVIDIA 495.44, you will get the `NVIDIA-Linux-x86_64-495.44.run`. Then run `sudo sh -c "$HOME/NVIDIA-Linux-x86_64-495.44.run -e"` for expert mode. Choose the things you needed from the install.
5. Once done, run `sudo mkinitrd` which calls dracut to regenerate the initramfs to make sure the NVIDIA modules are installed.
6. Disabling the nouveau can already be done when installing NVIDIA-Linux.XXX.xx.run. But in case you want to disable it manually, disable the nouveau driver by adding this to your kernel parameters `rd.driver.blacklist=nouveau nouveau.modeset=0` in `/etc/default/grub` or blacklisting it by running `# echo "blacklist nouveau" >> /etc/modprobe.d/nouveau-blacklist.conf`. If you did the former, run `sudo grub2-mkconfig -o /boot/grub/grub.cfg`.
7. Reboot.

##### Signing NVIDIA to enable Secure Boot (WIP)

### Hybrid Graphics/Optimus
1 See [Hybrid Graphics](hybrid_graphics.md)
