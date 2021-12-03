Using Linux on Lenovo Legion
============================

This is a compilation of instructions and resources for those who use a Linux
system (e.g. GNU plus Linux) on a Lenovo Legion laptop. More notes will be
added as I discover more things.

Contributing
------------

Any suggestion will be **very** appreciated. Feel free to open issues and pull
requests for suggestions, comments, and questions. If you have a workaround that
you do not see presented here, please, **please** either open an issue or pull
request. The goal here is to compile all the working solutions so everyone does
not have to spend countless hours digging for solutions online.

My system
---------

- **Machine**: Lenovo Legion 5 15ACH6H (Ryzen + NVIDIA)
- **Operating system**: Fedora 35
- **Linux kernel's version** (as of latest commit): 5.15.5

Table of Content
----------------

1. [Wi-Fi](#wi-fi)
	- [Downloading and installing the driver](#downloading-and-installing-the-driver)
	- [Kernel update](#kernel-update)
	- [Acknowledgment](#acknowledgment)
	- [TL;DR](#tldr)
2. [Brightness](#brightness)
	- [Proposed solutions](#proposed-solutions)
	- [Temporary solution](#temporary-solution)
	- [References](#references)

Wi-Fi
-----

This is perhaps the most frustrating issue and the problem you would want to fix
first. It is likely that your system does not have the driver for Realtek
RTL8852AE 802.11ax. Thus, you will need to download and install it.

### Downloading and installing the driver

1. Please **temporarily connect to the Internet using Ethernet or a wireless USB
   adapter.** You may also use another computer.
2. **Download/clone [this repository](https://github.com/lwfinger/rtw89).** You
   can clone any of the `main`, `v5`, `v6`, and `v7` branches. I tried the
   `main` branch and the `v7` branch, both of which worked (I am currently using
   the `v7`). But this might not be the case for your machine. In issue
   [#38](https://github.com/lwfinger/rtw89/issues/38) of the repository,
   GitHub user **lukinoway** insisted on using the `main` branch instead of the
   `v5` branch. Please try until you find one that works.
3. **Create a directory for storing a MOK** (Machine Owner Key) to allow the
   driver to run. You would want to keep this key, so store it with care. Assume
   you store them in `~/MOK`.
4. **Open a terminal in the directory you created in step 3.**
5. **Request a new MOK key pair.**

```bash
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=Custom MOK/"
```

6. **Import the MOK key.**

```bash
sudo mokutil --import MOK.der
```

7. **Reboot.** Upon seeing the splash screen with the Legion logo, you may see a
   blue screen (not of death) that asks if you want to manage MOK keys. **Select
   "Enroll MOK" on the menu and accept to enroll the new MOK key.** After this
   process, your OS will boot.
8. **Open a terminal in the `rtw89` directory that you cloned.**
9. **Compile the driver, sign the compiled driver files with the MOK key, and
   install the driver.**

```bash
make
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/MOK/MOK.priv ~/MOK/MOK.der rtw89core.ko
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/MOK/MOK.priv ~/MOK/MOK.der rtw89pci.ko
sudo make install
```

10. **Reboot.** Your device should be able to connect to wireless Wi-Fi networks
    now.

### Kernel update

After you have updated your Linux kernel, you need to recompile the driver and
install it again.

1. **Reboot.**
2. **Open the terminal in the `rtw89` directory.**
3. **Follow the steps below.** The last 4 commands is exactly the same as the
	 ones in step 9 of the previous section. Note that if you have deleted the MOK
	 key, you will have to complete steps 3-7 of the previous section. The
	 `make clean` command is very important.

```bash
git pull
sudo make uninstall
make clean
make
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/MOK/MOK.priv ~/MOK/MOK.der rtw89core.ko
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/MOK/MOK.priv ~/MOK/MOK.der rtw89pci.ko
sudo make install
```

### Acknowledgment

I thank GitHub user **lwfinger** and other contributors of the **rtw89** GitHub
repository for their work on the open-source driver. Many thanks to GitHub user
**lukinoway** for sharing how they installed the driver on Fedora 34.

### TL;DR

I do not recommend copying and pasting this snippet into your terminal. Follow
each step so you can fix any problem that arises.

```bash
# Connect using Ethernet or a wireless USB adapter to clone the rtw89 repository
# on GitHub. You can use another computer.

cd # Alternatively, cd to anywhere you want to clone the repository
git clone https://github.com/lwfinger/rtw89 # Use option -b to clone a specific branch

# Prepare a MOK for signing the driver.

mkdir ~/MOK
cd ~/MOK
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=Custom MOK/"
sudo mokutil --import MOK.der
reboot

# Upon reboot, you will see a blue screen for MOK management, after the Lenovo
# splash screen. Do not skip this blue screen. Use it to enroll the MOK key
# properly. After that, your Linux OS should boot.

cd # cd to where you cloned the directory
cd rtw89
make
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/MOK/MOK.priv ~/MOK/MOK.der rtw89core.ko
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/MOK/MOK.priv ~/MOK/MOK.der rtw89pci.ko
sudo make install
reboot
```

Brightness
----------

You might not be able to adjust screen brightness in your Linux OS on Lenovo
Legion. This might only happen in hybrid graphics mode.

### Proposed solutions

- **Try the `amdgpu.backlight=0` parameter in your kernel boot parameters.**
  When you boot your machine, press E on your keyboard when you see the GRUB
  boot menu. Then insert `amdgpu.backlight=0` into the kernel parameters. This
  addition is temporary so you can see if it works. If it does, you can add the
  parameter permanently e.g. using `grubby`:

```bash
sudo grubbby --args="amdgpu.backlight=0 --update-kernel $(sudo grubby --default-kernel)"
```

- The AMD iGPU might not support for brightness control due to a bug (refer to
  the _References_ section below). Thus, you can **try using the NVIDIA dGPU for
  this**. Please refer to the _References_ section below.

### Temporary solution

None of the above 2 proposed solutions have worked for me. Here is a temporary
solution:

```bash
xrandr --output MONITOR --brightness BRIGHTNESS
```

Replace `MONITOR` with the correct monitor identifier (try
`xrandr --listmonitors` for a list) and `BRIGHTNESS` with the appropriate
brightness from 0 to 1. Values over 1 are also accepted but not recommended.

For example, here is my output of `xrandr --listmonitors`:

```
Monitors: 2
 0: +*eDP 1920/344x1080/194+0+0  eDP
 1: +HDMI-1-0 2560/597x1440/336+1920+0  HDMI-1-0
```

I connect my laptop to an external monitor via HDMI, which is why you see 2
entries here (the first entry is obvious the Legion laptop's monitor). Hence, to
set the laptop's monitor's brightness to 0.5, I run:

```bash
xrandr --output eDP --brightness 0.5
```

This solution does not work very well with night light, however. As soon as
night light starts, you will lose what you have set for brightness. If you
attempt to set brightness using this method again, night light will turn off.

### References

Because I have not managed to fix this problem myself, the best I can do is to
provide resources I have come across that might be of interest. I will certainly
provide an update when I have managed to fix this problem.

- [**lenovo-legion5-15arh05-scripts**](https://github.com/antony-jr/lenovo-legion5-15arh05-scripts)
  (repository on GitHub). Please see inside the folders `AMDGPUFIX` and
  `XOrgConfigurationNvidia`. The repository contains many other useful fixes you
  might want for your Legion laptop, so I recommend checking the whole thing
  out. Many thanks to Antory Jr for this work.
- [This issue](https://gitlab.freedesktop.org/drm/amd/-/issues/1438) opened by
  Antony Jr.
