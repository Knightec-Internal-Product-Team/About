This document is not thoroughly tested yet.

# Introduction
There are two ways to flash this device. One is to connect it via micro USB to a host PC. This method requires a Linux native host, and you can follow the official instructions on Nvidias page then. Do not try with WSL or virtual machines or docker on Windows. It will not work, it's not supported, and many hours have been wasted trying to make it work.

# Windows
There is only one way that is known to be working, and that is flashing a jetpack image to an sd card.

1. Jetpack 5.1.3 is the latest Jetpack that is compatible with Xavier NX at the time of writing. Download the image for SD card here https://developer.nvidia.com/embedded/jetpack-sdk-513 (Orin NX is a newer platform, but since this is a Xavier make sure you pick the right one)
2. Flash the image to an SD card with Balena etcher. https://etcher.balena.io/

## Move the OS to NVME SSD
If the Xavier NX has an SSD attached to its m2 slot, you can move the OS to it since it's usually a bit faster than SD card.

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
```bash
sudo nano /boot/extlinux/extlinux.conf
```
Modify the `extlinux.conf` to make NVMe the default boot option:

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

Save and exit the file.

## Reconfigure the System for the New Root Filesystem on NVMe
Chroot into the NVMe drive environment:
```bash
sudo chroot /mnt/nvme
```
Update the `fstab` file in the NVMe drive:
```bash
nano /etc/fstab
```
Ensure an entry for the NVMe root filesystem is included:
```plaintext
/dev/nvme0n1p1  /  ext4  defaults  0  1
```
Exit the chroot environment:
```bash
exit
```

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

## Verify
After rebooting, verify that the system is running from the NVMe drive:
```bash
df -h /
```
The output should indicate that the root filesystem is `/dev/nvme0n1p1` or similar, confirming that the system is booting from the NVMe.

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

#### Restart the System:

To apply the changes, you need to restart the system. You can do this through SSH:

``` bash 
sudo reboot
```

### 4. Connect from a Remote Machine

On your remote machine, open your VNC client. Connect using the Jetson Nano's IP address. When prompted, enter the password you set earlier.
Conclusion

You should now be able to access your Jetson Nano's desktop remotely. If you face any issues, ensure your VNC client and server are compatible and that there are no firewall restrictions.

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

# Switch back to SD Card from NVME

If you for some reason want to switch back to booting from SD Card after you've switched to boot from NVME, follow this instruction

1. **Boot from the NVMe drive:**
   - Ensure your Jetson device is currently running from the NVMe drive.

2. **List your storage devices to identify the SD Card:
     ```bash
     lsblk
     ```
     **The rest of the instructions will assume that your SD Card is named mmcblk0p1**
3. **Edit the Boot Configuration:**
   - Open the `extlinux.conf` file:
     ```bash
     sudo nano /boot/extlinux/extlinux.conf
     ```
   - Find the `APPEND` line and update the `root` parameter to point to the SD card:
     ```plaintext
     APPEND ${cbootargs} console=ttyS0,115200n8 console=tty0 root=/dev/mmcblk0p1 rw rootwait
     ```
   - Save the file and exit the editor (`Ctrl+O` to save, `Enter` to confirm, and `Ctrl+X` to exit).

4. **Mount the SD Card Root Filesystem:**
   - Mount the SD card root filesystem to modify its configuration:
     ```bash
     sudo mount /dev/mmcblk0p1 /mnt
     ```

5. **Update the `/etc/fstab` File on the SD Card:**
   - Edit the `/etc/fstab` file on the SD card:
     ```bash
     sudo nano /mnt/etc/fstab
     ```
   - Ensure the root filesystem entry points to the SD card:
     ```plaintext
     /dev/mmcblk0p1  /  ext4  defaults  0  1
     ```
   - Save the file and exit the editor (`Ctrl+O`, `Enter`, `Ctrl+X`).

6. **Bind Mount Special Filesystems:**
   - Bind mount the special filesystems to ensure they are correctly handled:
     ```bash
     sudo mount --bind /proc /mnt/proc
     sudo mount --bind /sys /mnt/sys
     sudo mount --bind /dev /mnt/dev
     sudo mount --bind /run /mnt/run
     ```

7. **Chroot into the SD Card Environment:**
   - Chroot into the SD card environment:
     ```bash
     sudo chroot /mnt
     ```

8. **Verify Configuration Inside the Chroot:**
   - Verify that `/etc/fstab` is correctly configured inside the chroot environment:
     ```bash
     nano /etc/fstab
     ```
   - Ensure the root filesystem entry points to the SD card:
     ```plaintext
     /dev/mmcblk0p1  /  ext4  defaults  0  1
     ```
   - Save the file and exit the editor (`Ctrl+O`, `Enter`, `Ctrl+X`).

9. **Exit the Chroot Environment:**
   - Exit the chroot environment:
     ```bash
     exit
     ```

10. **Unmount Special Filesystems and the SD Card:**
   - Unmount the special filesystems and the SD card:
     ```bash
     sudo umount /mnt/proc
     sudo umount /mnt/sys
     sudo umount /mnt/dev
     sudo umount /mnt/run
     sudo umount /mnt
     ```

11. **Reboot Your Jetson Device:**
    - Now, reboot your Jetson device. It should boot from the SD card:
      ```bash
      sudo reboot
      ```

### Verification:
After rebooting, verify that your system is running from the SD card by checking the root filesystem:
```bash
df -h /
```
The output should indicate that the root filesystem is `/dev/mmcblk0p1` or similar, confirming that the system is booting from the SD card.
