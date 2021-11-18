## NVIDIA proprietary drivers

### Determine which driver is required by your GPU

There are two NVIDIA driver options available in the NVIDIA repository:

* The __x11-video-nvidiaG05__ package corresponds with the NVIDIA 460 series driver.
* The __x11-video-nvidiaG04__ package corresponds with the NVIDIA 390 series driver.

Please refer to the NVIDIA [website](https://www.nvidia.com/en-us/drivers/unix/) to determine which driver best supports your GPU.

### Setup the driver

#### With Yast
1. Open _YAsT2_.
2. Then _Software Management_.
3. On the menu, click __Configuration__ &gt; __Repositories__... (or do `Ctrl + R`).
4. Click __Add__ &gt; __Community Repositories__.
5. Select __nVidia Graphics Drivers__ &gt; __Accept__ &gt; __Trust__.
6. On the _Configured Software Repositories_ &gt; Click __Ok__
7. Back to _Software Management Windows_ &gt; __View__ &gt; __Repositories__ &gt; Select __nVidia Graphics Drivers__.
8. Select __x11-video-nvidiaG05__ &gt; __Accept__ (Some graphic cards need _G04_, see the first section above)
9. Reboot.

#### Using the command line
1. Add the NVIDIA Repository:
    - Tumbleweed
      ```
      sudo zypper addrepo --refresh https://download.nvidia.com/opensuse/tumbleweed NVIDIA
      ```
    - Leap
      ```
      sudo zypper addrepo --refresh 'https://download.nvidia.com/opensuse/leap/$releasever' NVIDIA
      ```
2. Install the appropriate driver by running `sudo zypper in x11-video-nvidiaG05` or `sudo zypper in x11-video-nvidiaG04`
3. Reboot.

#### CUDA
CUDA can be installed with the __nvidia-computeG05__ or __nvidia-computeG04__ package.

See [NVIDIA's documentation](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html) for further information.


### Hybrid Graphics/Optimus
See [Hybrid Graphics](hybrid_graphics.md)
