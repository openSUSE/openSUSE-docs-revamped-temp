# OpenSUSE: Pro Audio

Pro Audio generally refers to the art of recording, editing, and mixing high quality audio for the purpose of publication. Whether you are producing an album, a symphony, or a podcast, you will need to be familiar with *some* Pro Audio techniques.

Most users will not need to know or use more than a couple software programs to make simple edits or adjustments to voice recordings and music. This guide will cover these use cases and provide system hardware and software recommendations.

For more advanced users, setting up a Linux system for Pro Audio use is considered more difficult than on alternate Operating Systems. This guide will show you how to setup OpenSUSE Tumbleweed for advanced use cases.
  
# Hardware Recommendations:
- First and foremost if you are planning do achieve low latency while recording we recommend that you get a sound interface be it an USB, Firewire, or PCI card; your performance will be a lot better for that as opposed to using your motherboard's audio which its resources are shared by the system. 
# RAM
First, let us make sure that before we engage in setting up, our system is up to par. we recommend you have 16gb of ram or more  because it will give you the liberty of trying out more plugins that require more memory and it will mostlikely save you from low memory scenarios, though not common, its best to be safe than sorry.
# SSD
For your system & the music applications make sure you run them all in a solid state drive to reap the performance benefits. Also in your applictaions make sure to make use of other drives you are not using for "scratch" purposes; for volatile data; data that's temporary for the sake of space and performance; spread your workload.

*Let us get started in tuning the system then!*
___
# Get the codecs:
- Make sure your system is up to date first & install from & add <b>Packman Repository</b>:  
   `# zypper dup`  
   `# zypper in opi`  
   `opi codecs`  
# Add the Geekos DAW Repo (I use the priority of 90, which favours the packages from GeekosDAW repo)
- This is a crucial step in getting your audio workstation up to speed with plugins and scripts.  
 `# zypper ar -fp90 https://download.opensuse.org/repositories/home:/geekositalia:/daw/openSUSE_Tumbleweed/ Geekosdaw`  
# PulseAudio / Jack sync
- Make sure you install the required packages in order to use jack via pulseaudio
- **Note** that some packages here are a suggestion, but depending on your audio device, this may vary.   
 `# zypper in jack pulseaudio-module-jack alsa-plugins-pulse alsa-utils` 
- edit /etc/pulse/default.pa
1. find the lines that contain (do not uncomment, leave as is.):
> `#load-module module-alsa-sink`  
> `#load-module module-alsa-source device=hw:1,0`
2. underneath said line add:  
> **load-module module-jack-sink**  
 **load-module module-jack-source**
___
# Advanced tweaks:
This is where the fun begins, in here we will edit some of your system files for audio production using a text editor. You can use whatever editor you desire, in this case I'll use `nano` for example. 
**Required changes are in bold text.**

- Setup the system limits/priorities.  
`nano /etc/security/limits.conf`  
> output/partial output here  
**bolden changes**  
______
**[To be continued..]**
