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
- **Operating system**: Fedora 36
- **Windowing system**: X11
- **Linux kernel's version** (as of latest commit): `5.17.8-300.fc36.x86_64`

Table of Content
----------------

1. [Wi-Fi](#wi-fi)
	- [Downloading and installing the driver](#downloading-and-installing-the-driver)
	- [Kernel update](#kernel-update)
	- [Acknowledgment](#acknowledgment)
2. [Brightness](#brightness)
	- [Solutions](#solutions)
	- [Temporary solution](#temporary-solution)
	- [References](#references)
3. [Battery conservation mode](#battery-conservation-mode)
4. [Launch applications on dedicated graphics card](#launch-applications-on-dedicated-graphics-card)
5. [Keyboard's RGB](#keyboards-rgb)

Wi-Fi
-----

This is perhaps the most frustrating issue and the problem you would want to fix
first. It is likely that your system does not have the driver for Realtek
RTL8852AE 802.11ax. Thus, you will need to download and install it.

### Downloading and installing the driver

For this, we will work with the command line, so fire up the terminal. Logging
in as root is not necessary.

1. Please **temporarily connect to the Internet using Ethernet or a wireless USB
   adapter.** You may also use another computer;
2. **Download/clone [this repository](https://github.com/lwfinger/rtw89).** You
   can clone any of the `main`, `v5`, `v6`, and `v7` branches. I tried the
   `main` branch and the `v7` branch, both of which worked (I am currently using
   the `v7`);
3. **Create a directory for storing a MOK** (Machine Owner Key) to allow the
   driver to run. A good choice is `~/.MOK` (we will be using it for the rest
   of the instructions);

```bash
mkdir ~/.MOK
```

4. **Open the directory you created in step 3.**

```bash
cd ~/.MOK
```

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
8. **Open a terminal in the `rtw89` repository clone.** It is the directory you
   cloned the `rtw89` repository into in step 2.
9. **Compile the driver, sign the compiled driver files with the MOK key, and
   install the driver.**

```bash
make
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/.MOK/MOK.priv ~/.MOK/MOK.der rtw89core.ko
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/.MOK/MOK.priv ~/.MOK/MOK.der rtw89pci.ko
sudo make install
```

10. **Reboot.** Your device should be able to connect to wireless Wi-Fi networks
    now.

### Kernel update

After you have updated your Linux kernel, you need to recompile the driver and
install it again.

> **Note**: It has been a long time since I recompiled the driver after a Linux
> kernel update, so my suggestion is to follow the steps below only when you
> notice the Wi-Fi is not working after a kernel update.

1. **Reboot.** If you have already rebooted after updating the kernel, you may
   skip this step.
2. **Open the terminal in the `rtw89` directory.**
3. **Follow the steps below.** The last 4 commands is exactly the same as the
	 ones in step 9 of the previous section. Note that if you have deleted the MOK
	 key, you will have to complete steps 3-7 of the previous section.

```bash
git pull
sudo make uninstall
make clean
make
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/.MOK/MOK.priv ~/.MOK/MOK.der rtw89core.ko
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/.MOK/MOK.priv ~/.MOK/MOK.der rtw89pci.ko
sudo make install
```

4. **Reboot.**

A Bash script that automates this process can be found in [`src`](src/). Be sure
to modify the paths and file names accordingly.

### Acknowledgment

I thank GitHub user **lwfinger** and other contributors of the **rtw89** GitHub
repository for their work on the open-source driver. Many thanks to GitHub user
**lukinoway** for sharing how they installed the driver on Fedora 34.

Brightness
----------

You might not be able to adjust screen brightness in your Linux OS on Lenovo
Legion. This might only happen in hybrid graphics mode.

### Solutions

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

> **Update**: I used this temporary solution since none of the solutions above
> worked for me. As of now (using Fedora 36/X11/Linux kernel 5.17.8), however,
> I can control brightness in hybrid graphics mode normally using the function
> keys. I still have `amdgpu.backlight=0` in my kernel parameters; it is worth a
> try if you cannot adjust the brightness.

If none of the solution works for you, here is a temporary workaround:

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
entries here (the first entry is the Legion laptop's monitor). Hence, to set the
laptop's monitor's brightness to 0.5, I run:

```bash
xrandr --output eDP --brightness 0.5
```

This solution does not work very well with night light, however. As soon as
night light starts, you will lose what you have set for brightness. If you
attempt to set brightness using this method again, night light will turn off.

### References

- [**lenovo-legion5-15arh05-scripts**](https://github.com/antony-jr/lenovo-legion5-15arh05-scripts)
  (repository on GitHub). Please see inside the folders `AMDGPUFIX` and
  `XOrgConfigurationNvidia`. The repository contains many other useful fixes you
  might want for your Legion laptop, so I recommend checking the whole thing
  out. Many thanks to Antony Jr for this work.
- [This issue](https://gitlab.freedesktop.org/drm/amd/-/issues/1438) opened by
  Antony Jr.

Battery conservation mode
-------------------------

It is not a good idea to leave your laptop plugged in when it is already 100%
charged, as this will damage your laptop's battery's health. However, it is also
tiresome to have to manually unplug your laptop. Ideally, you want your laptop
to automatically stop the charge after the battery reaches a certain level
while it is still plugged in. Battery conservation does this. Windows users
enjoy enabling this feature in Lenovo Vantage. On Linux, simply run:

```bash
echo 1 > /sys/bus/platform/drivers/ideapad_acpi/VPC2004*/conservation_mode
```

When battery conservation mode is on, your battery will stop charging if it is
60% full or more. If you need to charge your laptop to, say, 80% so you can use
it unplugged later, you will have to disable battery conservation. To do so,
simply use the same command, replacing `1` with `0`.

Thanks again to Antony Jr.

Launch applications on dedicated graphics cards
-----------------------------------------------

> **Note**: Before you read this section, please have a dedicated graphics card
> driver installed.

Which graphics card is your machine running on?

```
glxinfo | egrep "OpenGL vendor|OpenGL renderer"
```

- If it is your dedicated graphics card (dGPU), I can think of 2 reasons (please
  contribute more, I am not an expert): 1. you have set the dGPU as your primary
  GPU; 2. you are in Legion's discrete graphics mode (configurable when boot).
  While this is totally fine if you know what you are doing, I find it overkill
  to have everything run on dGPU when I don't need it most of the time (consumes
  more power, increases your electricity bills);
- If it is your integrated graphics card (iGPU), this section is for you.

While in hybrid graphics mode, we are interested in launching certain
applications, such as Blender and video games, on our dGPU, to take advantage
of faster graphics computing power. GNOME has already allowed you to do so
by right clicking an application and selecting "Launch using Discrete Graphics
Card". But there is a more universal way to launch any application on the dGPU.

If you are using an AMD dGPU, you can start an application on the dGPU from the
terminal by setting the environment variable `DRI_PRIME`:

```
DRI_PRIME=1 command args...
```

And if you are using NVIDIA's:

```
__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia command args...
```

### Try it now

Start `glxgears`:

```
glxgears -info | grep GL_RENDERER
```

The program shows an animation of three intermeshing gears rotating. In the
terminal output, you should see your iGPU. This is mine (AMD iGPU, NVIDIA dGPU):

```
GL_RENDERER   = AMD Radeon Graphics (renoir, LLVM 14.0.0, DRM 3.48, 6.0.18-200.fc36.x86_64)
```

Kill the application. Now set the appropriate environment variable(s) and launch 
it again. So for example, I would use the following command:

```
__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia glxgears -info | grep GL_RENDERER
```

And the result:

```
GL_RENDERER   = NVIDIA GeForce RTX 3060 Laptop GPU/PCIe/SSE2
```

### Aliases

It would be a pain to have to set the variables every time. For convenience, put
this in your `.bashrc`, or wherever you set your Shell's configuration:

- AMD:
```
alias amd="DRI_PRIME=1"
```
- NVIDIA:
```
alias nvidia="__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia"
```

I can then start glxgears on my dGPU just by typing `nvidia glxgears`. Happy
computing!

Keyboard's RGB
--------------

GitHub user **4JX** created [an awesome cross-platform
application](https://github.com/4JX/L5P-Keyboard-RGB) for controlling the RGB
lights of Lenovo Legion laptops' keyboards. You can choose from one of the
presets (some of which require manual color setting) or even create your own
effects (please see the project on GitHub for more details). You can also set
the speed of the effect being used and the lights' brightness. The program can
be run in the CLI or the GUI. This works out of the box on my machine. Many
thanks again to GitHub user **4JX**.

![screenshot of the application](https://user-images.githubusercontent.com/24489228/145649411-944838e1-ed89-4a96-bd29-20138baa9707.png)
