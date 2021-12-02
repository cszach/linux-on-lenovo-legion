Using Linux on Lenovo Legion
============================

This is a compilation of instructions and resources for those who use a Linux
system (e.g. GNU plus Linux) on a Lenovo Legion laptop. More notes will be
added as I discover more things.

Contributing
------------

Any suggestion will be **very** appreciated. Feel free to open issues and pull
requests for suggestions, comments, and questions.

My system
---------

- Machine: Lenovo Legion 5 15ACH6H (Ryzen + NVIDIA)
- Operating system: Fedora 35
- Linux kernel's version (as of latest commit): 5.15.5

Table of Content
----------------

1. Wi-Fi
	- Downloading and install the driver
	- Kernel update
	- Acknowledgement

Wi-Fi
-----

This is perhaps the most frustrating issue and the problem you would want to fix
first. It is likely that your system does not have the driver for Realtek
RTL8852AE 802.11ax. Thus, you will need to download and install it.

### Downloading and installing the driver

1. Please temporarily connect to the Internet using Ethernet or a wireless USB
   adapter. You may also use another computer.
2. Download/clone [this repository](https://lwfinger/rtw89). You can clone any
   of the `main`, `v5`, `v6`, and `v7` branches. I tried the `main` branch and
   the `v7` branch, both of which worked (I am currently using the `v7`). But
   this might not be the case for your machine. In issue
   [#38](https://github.com/lwfinger/rtw89/issues/38) of the repository,
   GitHub user **lukinoway** insisted on using the `main` branch instead of the
   `v5` branch. Please try until you find one that works.
3. Create a directory for storing a MOK (Machine Owner Key) to allow the driver
   to run. You would want to keep this key, so store it with care. Assume you
   store them in `~/MOK`.
4. Open a terminal in the directory you created in step 3.
5. Request a new MOK key pair.

```bash
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=Custom MOK/"
```

6. Import the MOK key.

```bash
sudo mokutil --import MOK.der
```

7. Reboot. Upon seeing the splash screen with the Legion logo, you may see a
   blue screen (not of death) that asks if you want to manage MOK keys. Select
   "Enroll MOK" on the menu and accept to enroll the new MOK key. After this
   process, your OS will boot.
8. Open a terminal in the `rtw89` directory that you cloned.
9. Compile the driver, sign the compiled driver files with the MOK key, and
   install the driver.

```bash
make
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/MOK/MOK.priv ~/MOK/MOK.der rtw89core.ko
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/MOK/MOK.priv ~/MOK/MOK.der rtw89pci.ko
sudo make install
```

10. Reboot. Your device should be able to connect to wireless Wi-Fi networks now.

### Kernel update

After you have updated your Linux kernel, you need to recompile the driver and
install it again.

1. Open the terminal in the `rtw89` directory.
2. Follow the steps below. The last 4 commands is exactly the same as the ones
   in step 7 of the previous section. Note that if you have deleted the MOK key,
   you will have to complete steps 3-7 of the previous section. The `make clean`
   command is very important.

```
git pull
sudo make uninstall
make clean
make
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/MOK/MOK.priv ~/MOK/MOK.der rtw89core.ko
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ~/MOK/MOK.priv ~/MOK/MOK.der rtw89pci.ko
sudo make install
```

### Acknowledgement

I thank GitHub user **lwfinger** and other contributors of the **rtw89** GitHub
repository for their work on the driver. Many thanks to GitHub user
**lukinoway** for sharing how they installed the driver on Fedora 34.

