## Installing Ubuntu 14.04 on a MacBook Pro (11,1)

Your model version should be `MacBookPro 11,1`. You can verify this from OS X by following [Apple's steps here](http://support.apple.com/kb/ht4132), or by running `sudo dmidecode -s system-product-name` from Ubuntu.

You will need:

* A **USB flash drive** with at least 2GB of storage.
* Some sort of **wired connection**, at least briefly, to install wireless drivers. This can be Ethernet (through an Ethernet-to-USB adapter), or it could be USB tethering from a device connected to a mobile network.
* **These instructions**, either printed on paper (!), or visible from your phone or another laptop.

### Getting started (from OS X)

Prepare your hard drive by resizing the main OS X partition:

* Open up Disk Utility, click on the `____`, then click on the `Partition` tab.
* Drag the corner of the partition box to shrink the partition from 500GB to a size of your choice. Recommended: 100GB (leaving 400GB for Ubuntu).
* Click Resize `[...fill in with some more OS X instructions...]`

From the [Ubuntu Desktop download page](http://www.ubuntu.com/download/desktop/), download 14.04 LTS **64-bit**. Download the *normal* 64-bit ISO. Do **NOT** download the "64-bit Mac (AMD)" version.

Now we'll convert the `.iso` to the kind of `.img` file that Macs need to boot from.

`cd` to the directory where the `.iso` file lives, then run:

```bash
hdiutil convert -format UDRW -o ubuntu-14.04-desktop-amd64.img ubuntu-14.04-desktop-amd64.iso
```

OS X will actually output a `.img` with a `.dmg` extension added, so remove it:

```bash
mv ubuntu-14.04-desktop-amd64.img.dmg ubuntu-14.04-desktop-amd64.img
```

Insert the flash drive. Then find the identifier of the flash drive (e.g. `/dev/disk2`, by running:

```bash
diskutil list
```

Unmount the flash drive (replace `/dev/diskN` with the disk identifier you discovered above):

```bash
diskutil unmountDisk /dev/diskN
```

Then flash the `.img` to the drive. Use your disk identifier from above, but note the extra `r` that gets put in the middle.

```bash
sudo dd if=ubuntu-14.04-desktop-amd64.img of=/dev/rdiskN bs=1m
```

Eject the drive:

```bash
diskutil eject /dev/diskN
```

Then actually remove the drive.

**Note:** If you plan to use the `GSA-Guest` WiFi during the Ubuntu installation process, you may wish to sign in to the network from OS X before rebooting.

### Booting Ubuntu from USB

Reboot your computer, **and hold** the `Alt/Option` key during boot. The menu to select a boot device should appear.

You should see two hard-drive-like icons on the left, which are OS X and the OS X recovery partition. On the right, you should see 1 or 2 USB icons, labeled "EFI Boot".

If you see 2 USB icons, it shouldn't matter which one you pick. But if you want more peace of mind, you can unplug and replug in the USB drive, and only 1 should reappear.

Select the USB icon to enter into the Ubuntu GRUB selector. Pick **"Try Ubuntu without installing"**. Do *not* pick "Install Ubuntu" -- you will want to run some commands after installation but prior to rebooting.

### Installing (from fake Ubuntu)

Once inside the flash-booted Ubuntu, you will need to install wireless drivers from the network. This means you will need a **wired connection**. This could be Ethernet (using an adapter), or it could be over USB tethering (many Android phones, such as the [Nexus 5](https://play.google.com/store/devices/details?id=nexus_5_white_16gb) support this out-of-the-box).

Install drivers with:

```bash
sudo apt-get install bcmwl-kernel-source
```

The wireless should start working right away. If tethering, you may wish to disconnect the tethered device.

Start the installer from the "Install Ubuntu" desktop launcher. When asked how to install Ubuntu on the hard drive, select "Alongside Mac OS X".

Everything should install as normal. But when the install is done, choose "Keep Using Ubuntu". **Do not restart yet.**

You now have to fix the EFI boot order, so that Ubuntu is both available and the default.

Install the `efibootmgr` and run it:

```bash
sudo apt-get install efibootmgr
sudo efibootmgr
```

Ubuntu should be listed as `Boot0000`, and OS X as `Boot0080`. The `BootOrder` is likely set to `0080`, which means as it stands, it will boot directly to OS X.

Set the boot order to allow either one, and to default to Ubuntu, with:

```bash
sudo efibootmgr -o 0,80
```

Now you can safely reboot. Reboot back into Ubuntu -- you won't be able to boot back into OS X yet, which is something we'll fix from your real Ubuntu install.

### Post-installation fixes (from real Ubuntu)

You still need to do a couple things, from your real Ubuntu install.

First, you'll need to install those wireless drivers again:

```bash
sudo apt-get install bcmwl-kernel-source
```

Then we need to make two GRUB changes. The first adds a custom "MacOS" entry to the GRUB bootloader, which will let you properly boot into OS X:

```bash
sudo nano /etc/grub.d/40_custom
```

Add this to the end of the file (it exits GRUB, so that Mac's native bootloader picks up where it left off):

```
menuentry "MacOS" {
  exit
}
```

The second GRUB change fixes a reported occasional SSD freeze bug. (I've never seen it, but I'm following orders out of an abundance of caution.)

```bash
sudo nano /etc/default/grub
```

Change the `GRUB_CMDLINE_LINUX=""` line to read:

```
GRUB_CMDLINE_LINUX="libata.force=noncq"
```

Finally, save and update GRUB to make those two changes take effect:

```bash
sudo update-grub
```

### Dealing with the Retina display

You'll notice that the retina display makes everything tiny. Let's fix that.

Ubuntu has plans to better support high DPI monitors in future releases, but for now we'll need to work around it.

Increase the system font size:

```bash
gsettings set org.gnome.desktop.interface text-scaling-factor 1.5
```

Increase Firefox's pixel scaling. Visit `about:config`, and search for `"css"`. Find `layout.css.devPixelsPerPx` and change the value from `-1.0` (system default) to `2.0`.

Increase Chrome's default zoom. From the [Settings page](chrome://settings), click "Show advanced settings..." and scroll down to "Web content", then set the "Page zoom" to `175%`.

(Better support for HiDPI screens in Chrome on Linux [is coming](https://code.google.com/p/chromium/issues/detail?id=143619), but it depends on a Gtk+ replacement called Aura, which is currently [live in the Chrome dev channel](https://groups.google.com/a/chromium.org/forum/#!topic/chromium-dev/Zpu9801pPRc).)

Some things are still tiny, like the system tray. Such is life in Ubuntu on Macs, for the time being.

### Switching Alt and Cmd

If you're like me, you want the left-hand modifiers to go `Ctrl`/`Super`/`Alt`, and not `Ctrl`/`Alt`/`Super`, the way Mac keyboards default to.

To do that, make a file at `~/.xmodmap` that contains the following:

```
clear control
clear mod4
clear mod1

keycode 64 = Super_L
keycode 133 = Alt_L Meta_L

keycode 134 = Alt_R Meta_R
keycode 108 = Control_R

add mod4 = Super_L
add mod1 = Alt_L Meta_L
add mod1 = Alt_R Meta_R
add control = Control_L
add control = Control_R
```

You can test this out immediately by running `xmodmap ~/.xmodmap`, but this will only make it last until the next time you reboot.

To enable it every time you boot, create a file at `~/.config/autostart/switch_keys.desktop` with the following:

```
[Desktop Entry]
Name=Set Keyboard
Exec=xmodmap /path/to/.xmodmap
Terminal=false
Type=Application
```

**Note:** Change `/path/to/.xmodmap` above to the actual absolute path to your `.xmodmap` file. You can't use `$HOME`.

### Making Function keys work without Fn

By default, the Function keys (F1, F2, etc.) will do things like turn brightness up and down, and if you want to actually input F1 or F2, you need to hold down the Fn key. This applies to all 12 Function keys.

If you want to flip the behavior, create a new file:

```bash
sudo nano /etc/modprobe.d/hid-apple.conf
```

Whose body is the following one line:

```
options hid-apple fnmode=2
```

The next time you boot up, the behavior will be flipped, and you will need to hold down Fn to turn brightness up and down.

### Reference resources

I used these while figuring all this out:

* [Ubuntu official instructions for creating a bootable USB stick](http://www.ubuntu.com/download/desktop/create-a-usb-stick-on-mac-osx)
* [Ubuntu wiki instructions for 13.10/14.04 on MacBook Pro 11,1](https://help.ubuntu.com/community/MacBookPro11-1/Saucy)
* [Remapping your keyboard](http://www.chrisamiller.com/blog/posts/remapping-your-macbooks-keyboard-in-ubuntu-1204)
* [Running commands on startup](http://askubuntu.com/a/463280)
