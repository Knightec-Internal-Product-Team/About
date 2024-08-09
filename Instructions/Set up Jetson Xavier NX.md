This document is not thoroughly tested yet.

# Introduction
There are two ways to flash this device. One is to connect it via micro USB to a host PC. This method requires a Linux native host, and you can follow the official instructions on Nvidias page then. Do not try with WSL or virtual machines or docker on Windows. It will not work, it's not supported, and many hours have been wasted trying to make it work.

# Windows
There is only one way that is known to be working, and that is flashing a jetpack image to an sd card.

1. Jetpack 5.1.3 is the latest Jetpack that is compatible with Xavier NX at the time of writing. Download the image for SD card here https://developer.nvidia.com/embedded/jetpack-sdk-513 (Orin NX is a newer platform, but since this is a Xavier make sure you pick the right one)
2. Flash the image to an SD card with Balena etcher. https://etcher.balena.io/

The first time you start the jetson with the sd card, the boot some times get stuck on "starting configuration process" or similar. Then you need to make sure you connect a keyboard and mouse and restart the jetson. The GUI should come up after a couple of minutes and you can configure the OS.

## Move the OS to NVME SSD
If the Xavier NX has an SSD attached to its m2 slot, you can move the OS to it since it's usually a bit faster than SD card. This process can be done via SSH.

## Backup Your Data
Make sure to back up any important data on your SD card.

## Install an Editor
In this example, we'll use nano.
```bash
sudo apt-get install nano
```

## List Your Storage Devices to Identify the SD Card and NVMe
List your storage devices to identify the SD Card and NVMe:
```bash
lsblk
```
Assume the SD Card is `mmcblk0p1` and NVMe is `nvme0n1p1`, but it might be different on your device.

## Prepare the NVMe Drive
Boot your Jetson device using the SD card.
Open a terminal.
List your storage devices to identify the NVMe drive:
```bash
lsblk
```
Start `parted` to partition the NVMe drive (assuming it is `/dev/nvme0n1`):
```bash
sudo parted /dev/nvme0n1
```
Create a new partition table:
```parted
mklabel gpt
```
Create a new primary partition:
```parted
mkpart primary ext4 0% 100%
```
Exit `parted`:
```parted
quit
```

## Format the New Partition
Format the new partition with the ext4 filesystem and set a label (optional):
```bash
sudo mkfs.ext4 -L KnghtSSD /dev/nvme0n1p1
```

## Mount the NVMe Drive
Create a mount point and mount the NVMe drive:
```bash
sudo mkdir /mnt/nvme
sudo mount /dev/nvme0n1p1 /mnt/nvme
```

## Clone the SD Card to the NVMe Drive
Use `rsync` to copy the root filesystem, excluding the special filesystems:
```bash
sudo rsync -axHAWXS --numeric-ids --info=progress2 / /mnt/nvme
```

## Ensure Special Filesystems Are Not Mounted
Bind mount special filesystems to ensure they are correctly handled during the chroot:
```bash
sudo mount --bind /proc /mnt/nvme/proc
sudo mount --bind /sys /mnt/nvme/sys
sudo mount --bind /dev /mnt/nvme/dev
sudo mount --bind /run /mnt/nvme/run
```

## Update the Boot Configuration
Edit the boot configuration to make the NVMe drive the primary boot option:

First, save a backup.
```bash
sudo cp /boot/extlinux/extlinux.conf /boot/extlinux/extlinux.conf.bak
```
Then edit the file.
```bash
sudo nano /boot/extlinux/extlinux.conf
```
Modify the `extlinux.conf` to make NVMe the default boot option. You will see an entry with LABEL primary. Copy the section and rename it to nvme. Make sure that root is set to /dev/nvme0n1p1 on that section. Your end result will look something like this:

** Don't copy the text below , it's just an example. **
```plaintext
TIMEOUT 30
DEFAULT nvme

MENU TITLE Jetson Xavier NX Boot Options

LABEL nvme
  MENU LABEL Boot from NVMe (Primary)
  LINUX /boot/Image
  INITRD /boot/initrd
  APPEND ${cbootargs} root=/dev/nvme0n1p1 rw rootwait

LABEL sdcard
  MENU LABEL Boot from SD Card (Backup)
  LINUX /boot/Image
  INITRD /boot/initrd
  APPEND ${cbootargs} root=/dev/mmcblk0p1 rw rootwait
```
The information in your file is probably a little bit different, and that's why you make a copy of the section.
Pay attention to the MENU LABEL as this is what you will be seeing in your boot options menu.
Save and exit the file.

## Unmount and Reboot
Unmount the special filesystems and the NVMe drive:
```bash
sudo umount /mnt/nvme/proc
sudo umount /mnt/nvme/sys
sudo umount /mnt/nvme/dev
sudo umount /mnt/nvme/run
sudo umount /mnt/nvme
```
Reboot the Jetson device:
```bash
sudo reboot
```

During the boot up, you will see two choices, 0 and 1. It corresponds to the sections in your extlinux.conf, so if you put the nvme on top 0 is going to be booting from nvme. 

## Verify
After rebooting, verify that the system is running from the NVMe drive:
```bash
df -h /
```
The output should indicate that the root filesystem is `/dev/nvme0n1p1` or similar, confirming that the system is booting from the NVMe.

Now that you have booted into the NVME, you should copy the extlinux.conf from the sd card, since that file was changed after the file system was copied.

I understand your frustration. Copying the `extlinux.conf` from the SD card to the NVMe is a straightforward way to ensure that both boot options are preserved. Here’s how you can do it:

### Steps to Copy `extlinux.conf` from the SD Card to the NVMe

1. **Mount the SD Card:**
   - Assuming you're booted into the NVMe, mount the SD card to a directory:
   ```bash
   sudo mkdir /mnt/sdcard
   sudo mount /dev/mmcblk0p1 /mnt/sdcard
   ```

2. **Copy `extlinux.conf` from the SD Card to the NVMe:**
   - Copy the `extlinux.conf` file from the SD card's boot directory to the NVMe:
   ```bash
   sudo cp /mnt/sdcard/boot/extlinux/extlinux.conf /boot/extlinux/extlinux.conf
   ```

3. **Verify the Copy:**
   - Ensure that the file was copied correctly by viewing it:
   ```bash
   cat /boot/extlinux/extlinux.conf
   ```

4. **Unmount the SD Card:**
   - Once done, unmount the SD card:
   ```bash
   sudo umount /mnt/sdcard
   ```

5. **Reboot to Test:**
   - Reboot the system to confirm that the copied `extlinux.conf` works as expected:
   ```bash
   sudo reboot
   ```

# Summary
- The NVMe is configured as the primary boot drive, and the SD card is set as a backup boot option.
- The `extlinux.conf` file is modified to default to the NVMe drive, but provides an option to boot from the SD card if needed.
- If the NVMe drive fails, you can manually select the SD card from the boot menu to continue using the device.


# Set up VNC server (Untested)

Ubuntu on Jetson comes with Vino preinstalled.

### Configure

This guide will help you set up Vino, the GNOME default VNC server, on your Jetson Nano using SSH.

### Steps to set up Vino

You may need to start up the jetson with HDMI connected. (Verification needed). Replace YourPassword with the password of your choice. This will be used when you log in via VNC.

``` bash 
cd /usr/lib/systemd/user/graphical-session.target.wants && \
sudo ln -s ../vino-server.service ./.
```
If it says the service already exists that's fine.

``` bash 
gsettings set org.gnome.Vino prompt-enabled false && \
gsettings set org.gnome.Vino require-encryption false && \
gsettings set org.gnome.Vino authentication-methods "['vnc']" && \
gsettings set org.gnome.Vino vnc-password $(echo -n 'YourPassword' | base64)
```

### Auto Login (Needs testing)

#### Edit the GDM Configuration:

You will need to edit the GDM custom configuration file to enable automatic login for your desired user. Open the GDM custom configuration file with a text editor:

``` bash 

sudo nano /etc/gdm3/custom.conf
```

#### Add Configuration for Automatic Login:

Inside the custom.conf file, add the following lines to specify the user for automatic login:

``` bash 

[daemon]
AutomaticLoginEnable = true
AutomaticLogin = username
```

Replace username with the username of the account you want to log in automatically.

#### Save and Exit:

Press Ctrl+O, then press Enter, and finally press Ctrl+X to exit.

#### Configure virtual screen
You need to configure a virtual screen to get a good resolution on the VNC server.
``` bash 
sudo apt install nano
sudo cp /etc/X11/xorg.conf /etc/X11/xorg.conf.backup
sudo nano /etc/X11/xorg.conf
```
Paste this at the bottom of the file
``` bash 
Section “Screen”
    Identifier “Default Screen”
    Monitor “Configured Monitor”
    Device “Tegra0”
    SubSection “Display”
		Depth 24
		Virtual 1422 800 # Modify the resolution by editing these values
    EndSubSection
EndSection
```
Press CTRL + O and enter to save and CTRL + X to exit.

Be aware that this will make your HDMI stop working, because the screen gets assigned to the VNC instead. If you want to use the HDMI for display, you have to use SSH, remove the screen section from xorg.conf and reboot.

You can also skip the step where you add the screen section alltogether. Then you will have to start the jetson with HDMI connected, and you will be able to use VNC to see the screen.

#### Restart the System:

To apply the changes, you need to restart the system. You can do this through SSH:

``` bash 
sudo reboot
```

#### Restore xorg.conf (If things don't work)
If you want to restore the backup for some reason:
``` bash 
sudo cp /etc/X11/xorg.conf.backup /etc/X11/xorg.conf
```

### 4. Connect from a Remote Machine

On your remote machine, open your VNC client. Connect using the Jetson Nano's IP address. When prompted, enter the password you set earlier.
If it doesn't work, check if the service is running on the jetson:
```
systemctl --user status vino-server
```

If not, you have to run 
```
systemctl --user enable vino-server
systemctl --user start vino-server
```

Then it should work.

### Manually Adding Vino to Startup

To start the Vino server automatically, you can add it to the autostart applications in your desktop environment.

1. **Create a Desktop Entry for Vino**:

   ```bash
   mkdir -p ~/.config/autostart
   nano ~/.config/autostart/vino-server.desktop
   ```

2. **Add the Following Content to the File**:

   ```plaintext
   [Desktop Entry]
   Type=Application
   Name=Vino VNC Server
   Exec=/usr/lib/vino/vino-server
   X-GNOME-Autostart-enabled=true
   ```

3. **Save and Close the File**:
   Save the changes and exit the text editor.

4. **Reboot the System**:
   Reboot your system to apply the changes.

   ```bash
   reboot
   ```
