+++ 
draft = false
date = 2022-10-15T22:18:41+02:00
title = "Keychron K1 SE function keys not working on Linux"
slug = "keychron-se-function-keys-on-linux" 
+++
The function keys on Linux do not work by default on a [Keychron K1 SE](https://www.keychron.com/products/keychron-k1-se-wireless-mechanical-keyboard). 
Fixing that is thankfully rather easy.
<!--#more-->
As a owner of a Keychron K1 SE keyboard with hot swappable low profile brown Gateron switches and RGB backlight, 
it's rather critical that it works on both Windows and Linux. The keyboard is recognized as an Apple keyboard
on Linux, just like many other keyboards. The [Arch Linux wiki](https://wiki.archlinux.org/title/Apple_Keyboard) and [Ubuntu wiki](https://help.ubuntu.com/community/AppleKeyboard) have dedicated pages for this.
The TLDR to fix the functions keys not working is as follows:

1. Create a modprobe configuration file
```bash
$ sudo touch /etc/modprobe.d/hid_apple.conf
```
2. Edit the file and add the following content
```ini
options hid_apple fnmode=2
```
3. Regenerate the initramfs, which differs per distro. On Arch that would be
```bash
$ sudo mkinitcpio -P
```
or on Debian based
```batch
$ sudo update-initramfs -u
```

Reboot and make sure the keyboard is set to Windows/Android mode (physical switch on the back).