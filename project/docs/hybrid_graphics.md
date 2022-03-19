## Dual GPUs setups
This section is meant for users with dual 'graphic cards' (GPUs), often found in high-end laptops and essential to a pleasant video intensive experience (gaming, streaming, video editing, etc.)

On these hardware configurations, there is a software switch that picks the _integrated_ graphics processor for rendering most desktop applications, switching to the _dedicated_ GPU for more demanding tasks. The switch can be more or less fine-grained. Fine-grained switches, such as the ones used in _Kdenlive_ and _OBS Studio_, are able to use VAAPI and NVENC to offload specific processes of encoding/decoding to the dedicated GPU, while a coarse-grained switch would have to offload an entire program instead.

Some hardware configurations have a multiplexer (_mux_), a hardware switch that can be configured from the machine's UEFI to select the main graphics processor. A multiplexer is to be preferred if your hardware has one, as it performs better since there is no render offloading involved. This matters especially for users without external monitors, as the HDMI and DisplayPort outputs are usually connected directly to the dedicated GPU. Switching the multiplexer to the dedicated GPU or using an external monitor would thus offer better performance than using the built-in monitor or offloading the rendering to the dedicated GPU.

Update your system before proceeding to the next steps:
```
sudo zypper dup
```

### Dedicated NVIDIA GPU
To use an external monitor or be able to offload rendering to the NVIDIA GPU, you'll need to follow the next steps. The external monitor will work when you switch to the NVIDIA GPU. This section applies to users interested in the proprietary NVIDIA driver.

#### Getting the necessary software
1. [Install the correct Nvidia driver](install_proprietary.md).
2. Install the __suse-prime__ package using zypper or YaST. If you want to install via zypper run the following command in a terminal session: `sudo zypper in suse-prime`. Users on NVIDIA graphic cards released before GeForce 600 series should instead install __suse-prime-bbswitch__.
:   !!! info 
        In case you are using KDE, a widget called _SUSE Prime Selector_ can be added as a graphical way to switch graphics. You can install it with the command `sudo zypper in plasma5-applet-suse-prime`.
3. See the next sections according to your integrated and dedicated GPU combination. Alternatively, to offload rendering to the dedicated GPU (NVIDIA Optimus), see [Offloading specific applications to the NVIDIA GPU](hybrid_graphics.md#offloading-specific-applications-to-the-nvidia-gpu). This allows you to use both graphical units simultaneously.

### Intel and NVIDIA

Either one of these two options:

* From a terminal session: to switch to the NVIDIA GPU run `sudo prime-select nvidia`; conversely, for the Intel GPU run `sudo prime-select intel`. After executing either command, log out and back on to apply the changes.
* With the KDE widget _SUSE Prime Selector_, just click it and select _Switch to NVIDIA_. Then log out and back in and you should be set.

### AMD APU and NVIDIA
When using XOrg 1.20.6 or higher: to make the APU the primary GPU and have the ability to offload specific applications to the dedicated GPU, you can skip to the offloading section below.

If you are using an older release of XOrg, such as the case for Leap 15.2, or you wish to exclusively use the NVIDIA card, pick between the following steps:

* To enable only the NVIDIA GPU, run `sudo prime-select nvidia` from the command line.
* To enable the AMD GPU, run `sudo prime-select unset`.
    
After executing either command, log out and back on to apply the changes.

### Offloading specific applications to the Nvidia GPU

Warning: This section only applies to users with the proprietary driver and NVIDIA GeForce 600 series or higher utilizing the __nvidia-glG05__ driver package.

With the integrated Intel GPU or AMD APU set as your main GPU, you can also offload specific applications using an environment variable. For example, to run __supertuxkart__ using the NVIDIA driver, you can launch it from a terminal session as follows:
    
    __NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia __VK_LAYER_NV_optimus=NVIDIA_only supertuxkart
   
 
To offload a game from Steam using the NVIDIA driver, you can modify properties of the game, by right clicking it, going to **Properties** choosing __Set Launch Options__ and adding a line like this:

    __NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia __VK_LAYER_NV_optimus=NVIDIA_only %command%

For the offload process to work properly on integrated Intel cards, make sure the file `/etc/X11/xorg.conf.d/90-intel.conf` exists. If you do not have it, enable the NVIDIA card once and switch back to the integrated GPU; the required file will be created automatically along the way. Note that this only applies to XOrg sessions.

If everything worked out fine for you, feel free to copy and modify the _.desktop_ files of your more demanding programs (like MATLAB or Octave) and append the enviroment variables to the `Exec` section of the file:
    
    Exec=__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia __VK_LAYER_NV_optimus=NVIDIA_only /usr/bin/octave --gui %f
    
This makes the desired program offload to the dedicated GPU everytime you launch it from the shortcut.
        
### Dedicated AMD GPU or nouveau
To offload anything to the AMD GPU, simply use the special environment variable to launch the desired program, as per the following example:

    DRI_PRIME=1 supertuxkart

The same applies to users with the open source _nouveau_ driver for NVIDIA cards.
