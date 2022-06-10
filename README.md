![image](https://user-images.githubusercontent.com/76032/173039012-0e189b6d-f78d-4c9c-adad-e9daa3a185bf.png)

This repository contains a set of simple scripts and configuration files that help with the
configuration of your system when using an external Nvidia GPU.

My goal was to have a system that uses the Nvidia GPU for everything when it is plugged in,
but run Wayland on the integrated graphics whenever I am not using the eGPU (or when I am
using an AMD Radeon as an external GPU). I also wanted this to work entirely automatic.

These scripts were only tested on my ThinkPad X230 with an external graphics card connected
over the ExpressCard 2.0 slot, so your mileage may vary. I've only tested it on Debian (buster)
so far.

# Required software and configuration.
* GDM 3 and a line reading `WaylandEnable=false` or `#WaylandEnable=false` in `/etc/gdm3/daemon.conf`.
* Nvidia proprietary drivers must be installed.
* You must have a desktop session or Window manager that supports both Xorg and Wayland, or use
  different sessions depending on the plugged in GPU. If you want to use Xorg for both platforms,
  look into `contents/bin/nvidia-toggle-wayland` to remove the line that reconfigures GDM3,
  and only leave in the Xorg config file swapping.
* `Xorg.conf.nvidia` contains a hardcoded PCI bus ID. Change this to match your system.
  Use `nvidia-smi -q` to list Nvidia GPUs detected by the driver.

# Installation
1. Install GNU stow: `sudo apt install stow`
2. `sudo stow -t / -S contents && systemctl daemon-reload && systemctl enable nvidia-detect-config.service`

# Uninstall
1. From the directory where the repository is checked out, run `sudo stow -t / -D contents`.

# Notes
* There is no hotplug support whatsoever here. This is strictly a reconfigure-at-boot thing.
* Some Window managers do not execute scripts in `/etc/xdg/autostart/` at the start of a session.
  This will result in booting to a black screen when logging in with the Nvidia GPU connected.
  In such cases, log in to the window manager when running on the iGPU and configure it to run
  `/usr/bin/nvidia-offload` some other way.

# Optional configuration
* I tend to limit the power consumption of my Nvidia card since my power supply tethers right at
  the edge of good enough for the card. Doing this requires `nvidia-smi` and the commands to configure
  power consumption are commented out in `bin/nvidia-toggle-wayland`:

      # Tell the Nvidia driver to save power consumption configuration
      # This does *NOT* persist across reboots, only across Xorg restarts
      # or whatever other event triggers the Nvidia card to get suspended.
      nvidia-smi -pm 1
      # Limit the GPU power draw to 110W. This is the GPU only, and not the
      # full board power.
      nvidia-smi -pl 110

* I have compiled Debian packages for the Nvidia 390.67 long term support drivers. At the time of writing,
  this is two releases newer than the Debian testing/unstable packages (390.48). These packages are based
  on the code in https://salsa.debian.org/nvidia-team/nvidia-graphics-modules and the instructions at
  https://wiki.debian.org/NvidiaGraphicsDrivers#Building_newer_releases_from_SVN.
  To use these drivers, uncomment the repositories listed in `/etc/apt/source.list.d/nvidia-artisanal.list`,
  and run the following commands:

      sudo apt install apt-transport-https
      sudo apt-key adv --keyserver keyserver.ubuntu.com --recv 76A5B86C1DFA6542
      sudo apt update

  and update or install the `nvidia-driver`/`nvidia-driver-libs-i386` packages as normal. Should Debian
  update the drivers / libraries in the future, they should replace the packages I've built during an
  upgrade.
