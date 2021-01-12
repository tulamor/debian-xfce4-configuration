# Debian 10 Buster installatioin

1. Download latest version:  
https://www.debian.org/CD/http-ftp/#stable  
https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/    

2. Download firmware:  
Download the firmware archive for your platform and unpack it into a directory named firmware in the root of a removable storage device (USB/CD drive).  
When the installer starts, it will automatically find the firmware files in the directory on the removable storage and, if needed, install the firmware for your hardware.  
https://wiki.debian.org/Firmware  
http://cdimage.debian.org/cdimage/unofficial/non-free/firmware/  
OR:
Unofficial non-free images including firmware packages:  
https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/  
  
3. Create bootable USB drive  
https://www.debian.org/CD/faq/#write-usb  
Additionally to the method above for Linux systems, there is also the win32diskimager program available, which allows writing such bootable USB flash drives under Windows.  
Hint: win32diskimager will apparently only list input files named *.img by default, while the Debian images are named *.iso. Change the filter to *.* if you use this tool.  
https://www.debian.org/releases/jessie/i386/ch04s03.html.en - prepare bootable flash  
  
list of connected drives:  
`fdisk -l`

__Error:__ 
*Failed to determine the codename for the release debootstrap error*  
__Solution:__
While used unetbootin to create the image on a USB thumb drive. Unetbootin was not the problem, the media not being mounted under `/cdrom`. So it should be manually mounted the usb to `/cdrom`.  
Hit `ALT+F2` and then (if `ALT+F2` not working just load *ash*)
mount /dev/sdc1 /cdrom
then alt-f1 to get back to the install
The device for the usb drive may vary. For me it was /dev/sdc1.
The installer isn't smart enough to know to either mount the install media in /cdrom itself or look for it in `/target/media/cdrom`.

**************************************

`linux-image-amd64` is a generic metapackage, which depends on the specific default kernel package.  
In your particular case, `linux-image-amd64` probably depends on `linux-image-3.16-2-amd64`.  
In general it suffices to install the generic metapackage.  
You could alternatively install the specific `linux-image-3.16-2-amd64` package, but in general it is better style to install the generic meta package.

One specific advantage of installing the generic metapackage (and keeping it installed) is that it makes sure you always stay current on system upgrades.  
Otherwise, supposing you are upgrading from one Debian release to the next, or even from Debian stable to Debian testing, your kernel version will not automatically be upgraded, aside from minor Debian-specific upgrades for security reasons.  
However, if you have the generic metapackage installed, the latest kernel will be pulled in as a dependency.
___
## Tweaks

Install:
xfce4-whiskermenu-plugin - Alternate menu plugin for the Xfce desktop environment
amd64-microcode - Processor microcode firmware for AMD CPUs
OR
intel-microcode Processor microcode firmware for Intel CPUs
gufw - graphical firewall (Choose Deny on Incoming connection not Reject!)

Configure non-free repos, add "contrib non free":
```
deb http://deb.debian.org/debian/ buster main non-free contrib
```
```bash
sudo nano /etc/apt/sources.list
sudo apt update
sudo apt upgrade ### afterwards restart the system
```
Remember user at logon:
```bash
sudo nano /usr/share/lightdm/lightdm.conf.d/01_debian.conf
greeter-hide-users=false
```
Window Manager : Button layout > Remove unused buttons
Window Manager > Advanced : Windows snapping : Enable > To other windows

## Swapfile
(To create swap partition use `gparted`)  
To create swapfile:
```bash
sudo fallocate -l 10G /swapfile # 10 GB file created in the root dir
sudo chmod 600 /swapfile
sudo mkswap /swapfile           # format file to swap
sudo swapon /swapfile           # enable swap
sudo nano /etc/fstab            # make changes permanent
    #append: swapfile swap swap defaults 0 0
sudo free -h                    # check the status
```
To remove swapifle:
```bash
sudo swapoff /swapfile # deactivate
sudo rm /swapfile
sudo nano /etc/fstab   # remove swap entry
```
Swappiness defines how often swap will be used:
```bash
cat /proc/sys/vm/swappiness # default is 60
```
To change the default value:
```bash
sudo nano /etc/sysctl.conf 
    # append: vm.swappiness=10 # is the recommended value (if you have > 4 GB RAM)
```
## Hibernation
Determinate if swap is a file or a patition:  
`grep swap /etc/fstab`  
 If the output is `/swapfile` means it's located on the primary partition.  
 Get the UUID of the primary partition:  
`grep UUID /etc/fstab`  
Modify the grub to include the UUID, so the system will know the location of hibernate snapshot:  
`sudo nano /etc/default/grub`  
Edit line:  
`GRUB_CMDLINE_LINXU_DEFAULT="quiet splash resume=UUID={enter UUID code here instead of curly brackets}`  
The settings are done if the swap partition is used, for the swapfile continue:  
`sudo filefrag -v /swapfile`
```bash
Filesystem type is: ef53
File size of /swapfile is 10737418240 (2621440 blocks of 4096 bytes)
 ext:     logical_offset:        physical_offset: length:   expected: flags:
   0:        0..       0:    3201024..   3201024:      1:            
   1:        1..   10239:    3201025..   3211263:  10239:             unwritten
   2:    10240..   40959:    3241984..   3272703:  30720:    3211264: unwritten
```
here we need value - 3201024 to paste into grub:  
`sudo nano /etc/default/grub`  
`# append: GRUB_CMDLINE_LINXU_DEFAULT="quiet splash resume=UUID={} resume_offset=3201024`  
`sudo update-grub` # to update grub config file  
`sudo reboot`  

https://www.youtube.com/watch?v=NzUmsi9sgmg  


The Debian Administrator's Handbook:  
https://debian-handbook.info/browse/stable/sect.installation-steps.html  
  
`CTRL+LEFT ALT+F2` - Console Mode (`CTRL+LEFT ALT+F7` - GUI Mode)

Install grub boot loader  
```
apt-get purge xserver-xorg-legacy
```
# Configuration
Xfce is based on GTK+ version 2 (like Gnome 2). 
https://wiki.debian.org/Xfce

---------
XFCE's power managment GUI configurator <- to edit display power settings
--------
## Keyboard
Applications -> Settings -> Keyboard.  
Add:  `xfce4-terminal`

################# Add Keyboard Layout

Keyboard > Application shortcuts> Add "thunar ." > SUPER (WINDOWS) + E

---- Add shortcut to thunar

PATH: usr/bin/Thunar
```
echo $PATH:
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```
################# TO ADD KEYBOARD LANGUAGES:

https://wiki.debian.org/Keyboard

sudo nano /etc/default/keyboard
XKBLAYOUT="us,lt,ru"
XKBOPTIONS="grp:alt_shift_toggle"
sudo udevadm trigger --subsystem-match=input --action=change

----- PRINTSCREEN SHORTCUT:
XFCE Menu > Settings > Keyboard > Application Shortcuts
most programs are in /usr/bin , to select for keyboard shortcut use show all files
to find installation type in CLI: whereis <program name>
add the xfce4-screenshooter -r
MAN PAGE: https://docs.xfce.org/apps/screenshooter/usage

CTRL+ALT > firefox

### SCREENSHOTS

Add keyboard shortcut:  
`xfce4-screenshooter -wo screenshot`  
Add `screenshot` script to `/usr/bin/`:
```bash
date_time_ms=$(date +%F_%T_%3N)
cp "$1" /home/username/$date_time_ms.png
rm "$1"  
```

## Time

sudo dpkg-reconfigure tzdata
***if time configuration doesn't work mannualy:
sudo timedatectl set-ntp false

XFCE4 Issue: clock dissapears from toolbar:
Solution:
just set the custom format you want, even if it's the same as one of the built in formats.
The format specifiers are documented at docs.xfce.org/xfce/xfce4-panel/clock.
Set custom Format: %R
xfce4-panel --restart

## PulseAudio Volume Control

Panel preferences > Items > Hit + > Search for Pulseaudio

## Bluetooth
### Install driver for "Belkin F8T016"
```
sudo bash -c 'echo "blacklist hci_usb" >> /etc/modprobe.d/blacklist'
sudo bash -c 'echo "hci_usb reset=1" >> /etc/modules'
```
The first edit stops automatically loading the module and the second loads the module with the correct parameter.
```
sudo apt-get install pulseaudio-module-bluetooth
killall pulseaudio
```
Problem:  
*Connection Failed: blueman.bluez.errors.DBusFailedError: Protocol not available.*

Bluetooth installing:

```bash
pactl list short | grep module-bluetooth
apt policy pulseaudio-module-bluetooth
sudo apt-get install pulseaudio-module-bluetooth
pactl load-module module-bluetooth-discover
```

If it doesn't work, try restarting pulseaudio:

```bash
pulseaudio -k
pulseaudio -D
```

```bash
sudo apt install pulseaudio-module-bluetooth
pulseaudio -k
pulseaudio --start
sudo apt-get install bluez-tools
run: bluetoothctl
scan on
trust   [DEVNAME]
pair    [DEVNAME]
connect [DEVNAME]
```

Change Bluetooth Audio profile:

```
systemctl restart bluetooth
pacmd list-cards
pacmd set-card-profile {index} {profile_name}
pacmd set-card-profile 3 a2dp_sink
```

Problem:  
*blueman.bluez.errors.DBusFailedError: Resource temporarily unavailable*  
Solution:  
`sudo systemctl restart bluetooth`  
scan for the device once again, pair and connect

___
#### SCP USE:
http://www.hypexr.org/linux_scp_help.php
1. On local machine run script to take file from dev machine:  
    scp username@machinename.cern.ch:/build/source/filename /home/destination_folder/
2. Upload file from local machine to server:  
    scp /home/source username@machinename.cern.ch:/build/destination_folder/  
    Example with ip adress:  
    scp file.txt remote_username@10.10.0.2:/remote/directory

To specify name of the file while will be uploaded to server:  
    scp file.txt remote_username@10.10.0.2:/remote/directory/newfilename.txt

#To copy a directory from a local to remote system, use the -r option:
    scp -r /local/directory remote_username@10.10.0.2:/remote/directory
More about scp commands: https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/

########## Software installation:
sudo apt-get install chromium
https://www.howtogeek.com/100361/how-to-optimize-google-chrome-for-maximum-privacy/
https://www.ghacks.net/2013/11/09/disable-privacy-sensitive-features-google-chrome/

Install VSCODE:
sudo apt install ./<file>.deb
turn off telemetry:
https://code.visualstudio.com/docs/supporting/faq#_how-to-disable-telemetry-reporting
https://code.visualstudio.com/docs/python/python-tutorial
install pylint for python3:
sudo apt install python3-pip

add pylint to PATH:  
sudo nano /etc/profile  
add pylint location  
source /etc/profile  
sudo apt-get terminator  
sudo apt-get install vlc  
sudo apt install kolourpaint4 (better use Pinta but it crashes, need to wait for bug fix)  
obs-studio  
audacity  
losslessCutd  
  
******** SPECIFICATION
Short specs:  
lspci  
00:02.0 VGA compatible controller: Intel Corporation 2nd Generation Core Processor Family Integrated Graphics Controller (rev 09)
Full specs:  
lspci -v

---- Find video card information:
install lshw  
sudo lshw -c video  

********

## REMOVE PACKAGE COMPLETELY:
sudo apt-get purge wordpress
sudo apt-get autoremove

---- Install nvidia drivers
---- Install nvidia-xconfig
writes file to:
/etc/X11/xorg.conf

-------------
$USERNAME is not in the sudoers file.  This incident will be reported
You can check if the currently logged in user belongs to the sudo group by using the groups command
If the groups command does not return sudo on Debian-based Linux distributions, then that username can't run commands with sudo
So to get root, then add your user to the sudo group, use:
su -
usermod -aG sudo YOUR_USERNAME
exit

Where:

    su switches to the root user, while - runs a login shell so things like /etc/profile, .bashrc, and so on are executed (this way commands like usermod will be in your $PATH, so you don't have to type the full path to the executable). You may also use sudo su - instead of su -
    You need to replace YOUR_USERNAME with the username that you want to add to the sudo group.
    I have used usermode to add a group to an existing user because it should work on any Linux distribution. adduser or useradd can also be used for this (adduser USERNAME -G sudo) but they may not work across all Linux distributions. Even though this article is for Debian, I wanted to make it possible to use this on other Linux distributions as well (I noticed that adduser doesn't work on Solus OS for example).
    exit exists the root shell, so you can run commands as a regular user again
After this, sudo still won't work! You will need to logout from that user, then relogin, and sudo will work.

Another typical example is to allow the user to run only specific commands via sudo. For example, to allow only the mkdir and rmdir commands you would use:
/etc/sudoers

username ALL=(ALL) NOPASSWD:/bin/mkdir,/bin/rmdir

Instead of editing the sudoers file, you can achieve the same by creating a new file with the authorization rules in the /etc/sudoers.d directory. Add the same rule as you would add to the sudoers file:

echo "username  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/username

This approach makes the management of the sudo privileges more maintainable. The name of the file not important, but it is a common practice to name the file according to the username.

**************************************
Completely remove app with it's deps
sudo apt-get purge --auto-remove <package_name>
**************************************

find against freezing SSHFS:
unmount the drive:
fusermount -u -z /media/name_of_drive

sshfs -odebug,sshfs_debug,loglevel=debug
sshfs User@Server:/ /media/Server -o debug,sshfs_debug,loglevel=debug port=10000,allow_other,default_permissions

umount /mount/point -f

Also, like above, don't autocomplete path with TAB. This "fix" made all my hung apps and windows complete their task queues and work as normal.

sshfs <email address hidden>:/username /mount/dir -o idmap=user -o allow_other -o reconnect


Workaround:
  
If you go to a different TTY (ctrl+alt+F1) in stead of a Terminal Emulator, tab-complete will have no problem. You can
umount --force /mount/point
It will fail, but repeat a bunch of times and it will unmount.
Switch back you your X session and all your resources are released and functional.
---------
Solution:
* configure ssh with a ServerAliveInterval value. When there's no traffic to the ssh server, the ssh client will check if the server is still alive every once in a while. This will keepp traffic alive which means that whatever is in between will not terminate the connection. You will want this value to be small but not so small that it sends packagesevery second. For me, 120 (2 minutes) solves the problems but others may need to set it at 30. Add the following to /etc/ssh/ssh_config or to ~/.ssh/config file
  
ServerAliveInterval 120

----
sudo umount -l /mnt/mysshfs" 

for debbuging purpose use sshfs with options:
-o debug
-o sshfs_debug


Solution for hanging SSHFS:

Use -o reconnect,ServerAliveInterval=15,ServerAliveCountMax=3

The combination ServerAliveInterval=15,ServerAliveCountMax=3 causes the I/O errors to pop out after one minute of network outage. This is important but largely undocumented. If ServerAliveInterval option is left at default (so without the alive check), processes which experience I/O hang seem to sleep indefinitely, even after the sshfs gets reconnect'ed. I regard this a useless behavior.

On -o reconnect without assigning ServerAliveInterval is that any I/O will either succeed, or hang the application indefinitely if the ssh reconnects underneath. A typical application becomes entirely hung as a result. If you'd wish to allow I/O to return an error and resume the application, you need ServerAliveInterval=1 or greater.

**************************************

### XFCE Shortcuts

CTRL ALT D =  Hide all windows  
ALT F3     =  Application finder [ALT + F2]  
Windows L  =  Logout window  
ALT F10    =  MAXIMIZE  
Super P    =  XF86Display  
  
**************************************
  
Get motherboard model:  
sudo dmidecode -t 2  

**************************************
  
cat /proc/version - Installed Linux version information  
/usr/sbin/apache2 -v - Apache version information  
  
**************************************
  
/etc/init.d/apache2 stop  
/etc/init.d/apache2 start  
  
**************************************
  
Format Hard Drive  
mkfs.ext3 /dev/sda  
  
**************************************
 
List all of the partitions on all devices that are listed in /proc/partitions:  
sudo fdisk -l  
List all block storage devices:  
lsblk  
  
**************************************
View information about connections:  
`cd /etc/NetworkManager/system-connections & ls -l`  
Debug wifi connection:  
`dmesg`

Enabling and disabling wireless devices:

```bash
rfkill list  
rfkill unblock wlan  
ip -brief link
```

**************************************

### Configure multiple displays
```
cd /usr/share/X11/xorg.conf.d  
sudo nano ~/.config/xfce4/xfconf/xfce-perchannel-xml/displays.xml  
/etc/X11/xorg.conf  
```
1)-----  
gtf 1920 1080 60  
OUTPUT:  
1920x1080 @ 60.00 Hz (GTF) hsync: 67.08 kHz; pclk: 172.80 MHz  
Modeline "1920x1080_60.00"  172.80  1920 2040 2248 2576  1080 1081 1084 1118  -HSync +Vsync  
2)-----  
xrandr --newmode "1920x1080_60.00"  172.80  1920 2040 2248 2576  1080 1081 1084 1118  -HSync +Vsync  
xrandr --newmode "1080p"  172.80  1920 2040 2248 2576  1080 1081 1084 1118  -HSync +Vsync  
3)-----  
xrandr --addmode DVI-I-1 1920x1080_60.00  
xrandr --addmode DVI-I-2 1080p  
4)-----  
xrandr --output DVI-I-1 --mode 1920x1080_60.00 --rate 60  
xrandr --output DVI-I-2 --mode 1080p  
  

How to know vnc port listening current session:
One option if you have ssh connection to the other machine is to find the litening ports for vnc as explained at the end of this post
You could login a ssh session and find out the number by
    netstat -tulpn | grep vnc

and you will get something like the following

tcp   0    0 127.0.0.1:5910     0.0.0.0:*     LISTEN      5365/Xvnc

and then you know 5910 was the port you connected to.


apt-cache search <-- search available packages

Use gparted for disk resize

---

## Mount android device via mtp

```bash
jmtpfs -l
mkdir android
jmtpfs /media/android
fusermount -u ~/android
```