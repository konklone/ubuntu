## Installing Ubuntu 15.10 on a MacBook Pro (11,1)

This tutorial will create a **fully disk encrypted** Macbook running **only Ubuntu**. This will **completely remove your OS X partition**.

To install _Debian_ on a Macbook Pro, check out [Jessie Frazelle's tutorial](https://blog.jessfraz.com/post/linux-on-mac/).

Your model version should be `MacBookPro 11,1`. You can verify this from OS X by following [Apple's steps here](http://support.apple.com/kb/ht4132), or by running `sudo dmidecode -s system-product-name` from Ubuntu.

You will need:

* A **USB flash drive** with at least 2GB of storage.
* Some way to download/transfer WiFi drivers after installation. The easiest way is to use a tethered network connection over USB.

You can create the Ubuntu boot USB stick [from OS X](#starting-from-os-x), or you can do it [from Ubuntu](#starting-from-ubuntu).

### If You're Starting from OS X

From the [Ubuntu Desktop download page](http://www.ubuntu.com/download/desktop/), download 16.04 **64-bit**. Download the *normal* 64-bit ISO. Do **NOT** download the "64-bit Mac (AMD)" version.

Now we'll convert the `.iso` to the kind of `.img` file that Macs need to boot from.

`cd` to the directory where the `.iso` file lives, then run:

```bash
hdiutil convert -format UDRW -o ubuntu-16.04-desktop-amd64.img ubuntu-16.04-desktop-amd64.iso
```

OS X will actually output a `.img` with a `.dmg` extension added, so remove it:

```bash
mv ubuntu-16.04-desktop-amd64.img.dmg ubuntu-16.04-desktop-amd64.img
```

Insert the flash drive. Then find the identifier of the flash drive (e.g. `/dev/disk2`), by running:

```bash
diskutil list
```

Unmount the flash drive (replace `/dev/diskN` with the disk identifier you discovered above):

```bash
diskutil unmountDisk /dev/diskN
```

Then flash the `.img` to the drive. Use your disk identifier from above, but note the extra `r` that gets put in the middle.

```bash
sudo dd if=ubuntu-16.04-desktop-amd64.img of=/dev/rdiskN bs=1m
```

Eject the drive:

```bash
diskutil eject /dev/diskN
```

Then actually remove the drive.

### If You're Starting from Ubuntu

From the [Ubuntu Desktop download page](http://www.ubuntu.com/download/desktop/), download 16.04 **64-bit**. Download the *normal* 64-bit ISO. Do **NOT** download the "64-bit Mac (AMD)" version.

Figure out the correct device identifier for the USB drive. It may be `/dev/sdc`, or `/dev/sdd`, etc. If you already run Ubuntu, I'm trusting you to figure out how to identify the USB stick's identifier.

Create a USB stick by running the following command, replacing `/dev/sdX` with the device identifier you found:

```bash
sudo dd if=ubuntu-16.04-desktop-amd64.iso of=/dev/sdX
```

### Booting Ubuntu from USB

Reboot your computer, **and hold** the `Alt/Option` key during boot. The menu to select a boot device should appear.

You should see two hard-drive-like icons on the left, which are OS X and the OS X recovery partition. On the right, you should see 1 or 2 USB icons, labeled "EFI Boot".

If you see 2 USB icons, it shouldn't matter which one you pick. But if you want more peace of mind, you can unplug and replug in the USB drive, and only 1 should reappear.

Select the USB icon to enter into the Ubuntu GRUB selector. Pick **"Try Ubuntu without installing"**.

### Installing (from fake Ubuntu)

Once inside the flash-booted Ubuntu, you may notice that the screen is not well-scaled for the high-density screen. You can fix that if you want by [following the instructions below](#high-density-display), but you'll have to do it again after install anyway.

Wireless drivers ship with the boot image, but aren't installed by default. Install them with:

```bash
sudo apt-get install bcmwl-kernel-source
```

The wireless should start working right away.

Start the installer from the "Install Ubuntu" desktop launcher. When asked how to install Ubuntu on the hard drive, elect to erase all operating systems and install Ubuntu.

When installing Ubuntu, pick the option to encrypt your hard drive during install. This will auto-select an option about LVM as well, which is fine.

During installation, select the option to wipe the entire drive, even unused blocks -- even though the installer says this may make installation take much longer. It won't take that long.

### Post-installation fixes (from real Ubuntu)

#### Fix the wireless

The wireless drivers ship with the boot image, but aren't _installed_ by the boot image, so you need to either get a USB network connection to your laptop to install them, or you need to manually download and install 2 `.deb` files to proceed.

If you have a USB tethered network connection, use it and install those wireless drivers again:

```bash
sudo apt-get install bcmwl-kernel-source
```

If you don't have a tethered network connection, download these two files from some other computer:

* [`dkms` for Ubuntu 15.10](http://packages.ubuntu.com/xenial/all/dkms/download)
* [`bcmwl-kernel-source` for Ubuntu 15.10](http://packages.ubuntu.com/xenial/amd64/bcmwl-kernel-source/download)

Transfer the two `.deb` files via USB to your Macbook, then install them with `dpkg`:

```
sudo dpkg -i dkms_2.2.0.3-2ubuntu6_all.deb
sudo dpkg -i bcmwl-kernel-source_6.30.223.248+bdcom-0ubuntu8_amd64.deb
```

#### High density display

You'll notice that the retina display makes everything tiny. Fix this by going to your `System Settings`, then to `Displays`. Change the "Scale for menu and title bars" from `1` to `1.75`.

![scaling display](images/scale.png)

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

Then update the boot image:

```
sudo update-initramfs -u -k all
```

The next time you boot up, the behavior will be flipped, and you will need to hold down Fn to turn brightness up and down.

### Other Ubuntu things to do

* [Enable recognition of your U2F FIDO key](https://konklone.com/post/get-a-fido-key-right-now-and-log-into-stuff-with-it#getting-it-working-on-linux).

### Reference resources

I used these while figuring all this out:

* [Ubuntu official instructions for creating a bootable USB stick](http://www.ubuntu.com/download/desktop/create-a-usb-stick-on-mac-osx)
* [Ubuntu wiki instructions for 13.10/14.04 on MacBook Pro 11,1](https://help.ubuntu.com/community/MacBookPro11-1/Saucy)
* [Ubuntu wiki instructions for 14.10 on Macbook Pro 11,1](https://help.ubuntu.com/community/MacBookPro11-1/utopic)
* [Remapping your keyboard](http://www.chrisamiller.com/blog/posts/remapping-your-macbooks-keyboard-in-ubuntu-1204)
* [Running commands on startup](https://askubuntu.com/a/463280)
