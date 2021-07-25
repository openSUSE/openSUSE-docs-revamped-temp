# OpenSUSE: Pro Audio
Setting up Pro Audio can be tricky, it takes understanding your hardware and workflow but, in this guide ill show you how I setup using OpenSUSE Tumbleweed, a few tools, and a common understanding of what I will be doing.
  
# Hardware Recommendations:
- First and foremost if you are planning to do low latency audio/voice recording, it is **imperative** that you get a sound interface be it an USB, Firewire, or PCI card; your performance will be miles better for that as opposed to using your motherboard's audio. 
# RAM
First, let us make sure that before we engage in setting up, our system is up to par. I recommend you to have 16gb of ram or more simply because it will give you the liberty of trying out certain plugins that require more memory and it will mostlikely save you from low memory scenarios, though not common, its best to be safe than sorry.
# SSD
For your system & the music applications make sure you run them all in a solid state drive to reap the performance benefits. Also in your applictaions make sure to make use of other drives you are not using for "scratch" purposes; for volatile data; data that's temporary for the sake of space and performance; spread your workload.

*Let us get started in tuning the system then!*
___
# Get the codecs:
Make sure your system is up to date first & installfrom & add <b>Packman Repository</b>:
  > `# zypper dup`  
  > `# zypper in opi`  
  > `opi codecs`  
# Add the Geekos DAW Repo (I use the priority of 90, which favours the packages from GeekosDAW repo)
- This is a crucial step in getting your audio workstation up to speed with plugins and scripts.
> `# zypper ar -fp90 https://download.opensuse.org/repositories/home:/geekositalia:/daw/openSUSE_Tumbleweed/ Geekosdaw`  
# PulseAudio / Jack sync
- Make sure you install the required packages in order to use jack via pulseaudio
- **Note** that some packages here are a suggestion, but depending on your audio device, this may vary. 
> `# zypper in jack pulseaudio-module-jack alsa-plugins-pulse alsa-utils` 
- edit /etc/pulse/default.pa
1. find the lines that contain (do not uncomment, leave as is.):
> `#load-module module-alsa-sink`  
> `#load-module module-alsa-source device=hw:1,0`
2. underneath said line add:
> `load-module module-jack-sink`  
> `load-module module-jack-source`

______
**[To be continued..]**
