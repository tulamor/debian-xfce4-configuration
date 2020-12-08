Debian 10 Buster

1. Download latest version: 
https://www.debian.org/CD/http-ftp/#stable
Download latest version: https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/
(The netinst CD here is a small CD image that contains just the core Debian installer code and a small core set of text-mode programs)


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

list of connected drives fdisk -l
I've got an error, so here is the solution:

failed to determine the codename for the release debootstrap error

I used unetbootin to create the image on a USB thumb drive.
unetbootin was not the problem. The problem was related to the media not being mounted under /cdrom
I had to manually mount the usb to /cdrom and when I retried the install, I was able to get past the "bootstrap error".
I did this by hitting alt-f2 and then (if alt-f2 not working just load ash)
mount /dev/sdc1 /cdrom
then alt-f1 to get back to the install

The device for the usb drive may vary. For me it was /dev/sdc1.
I have no idea why the installer isn't smart enough to know to either mount the install media in /cdrom itself of look for it in /target/media/cdrom where it already had it mounted.



**************************************

linux-image-amd64 is a generic metapackage, which depends on the specific default kernel package. 
In your particular case, linux-image-amd64 probably depends on linux-image-3.16-2-amd64. 
In general it suffices to install the generic metapackage. 
You could alternatively install the specific linux-image-3.16-2-amd64 package, but in general it is better style to install the generic meta package.

One specific advantage of installing the generic metapackage (and keeping it installed) is that it makes sure you always stay current on system upgrades. 
Otherwise, supposing you are upgrading from one Debian release to the next, or even from Debian stable to Debian testing, your kernel version will not automatically be upgraded, aside from minor Debian-specific upgrades for security reasons. 
However, if you have the generic metapackage installed, the latest kernel will be pulled in as a dependency.

**************************************

https://debian-handbook.info/browse/stable/sect.installation-steps.html
https://www.debian.org/releases/stable/arm64/ch06s01.html.en

Ctrl+Left Alt+F2- console mode
install grub boot loader

apt-get purge xserver-xorg-legacy
apt-get install xrdp

Applications -> Settings -> Keyboard.
Open the Application Shortcuts Tab. This is what it looks like(customised).
To Add a new Shortcut, click on Add 
xfce4-terminal

How to know vnc port listening current session:
One option if you have ssh connection to the other machine is to find the litening ports for vnc as explained at the end of this post
You could login a ssh session and find out the number by
    netstat -tulpn | grep vnc

and you will get something like the following

tcp   0    0 127.0.0.1:5910     0.0.0.0:*     LISTEN      5365/Xvnc

and then you know 5910 was the port you connected to.

################
# Configuration
Xfce is based on GTK+ version 2 (like Gnome 2). 
https://wiki.debian.org/Xfce

---------
XFCE's power managment GUI configurator <- to edit display power settings
--------

################# Add Keyboard Layout

Keyboard > Application shortcuts> Add "thunar ." > SUPER (WINDOWS) + E

---- Add shortcut to thunar

PATH: usr/bin/Thunar
echo $PATH:
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

----- PRINTSCREEN SHORTCUT:
XFCE Menu > Settings > Keyboard > Application Shortcuts
most programs are in /usr/bin , to select for keyboard shortcut use show all files
to find installation type in CLI: whereis <program name>
add the xfce4-screenshooter -r
MAN PAGE: https://docs.xfce.org/apps/screenshooter/usage

CTRL+ALT > firefox

################# CONFIGURE TIME (DEBIAN)

sudo dpkg-reconfigure tzdata
***if time configuration doesn't work mannualy:
sudo timedatectl set-ntp false

###################

XFCE4 Issue: clock dissapears from toolbar:
Solution:
just set the custom format you want, even if it's the same as one of the built in formats.
The format specifiers are documented at docs.xfce.org/xfce/xfce4-panel/clock.
Set custom Format: %R
xfce4-panel --restart

################# TO ADD AUDIO CONTROL ON PANEL:

Panel preferences > Items > Hit + > Search for Pulse audio

################# TO ADD KEYBOARD LANGUAGES:

https://wiki.debian.org/Keyboard

sudo nano /etc/default/keyboard
XKBLAYOUT="us,lt,ru"
XKBOPTIONS="grp:alt_shift_toggle"
sudo udevadm trigger --subsystem-match=input --action=change

#################
bluetooth for any pc
pactl list short | grep module-bluetooth
apt policy pulseaudio-module-bluetooth
sudo apt-get install pulseaudio-module-bluetooth
pactl load-module module-bluetooth-discover

If it doesn't work, try restarting pulseaudio:
pulseaudio -k
pulseaudio -D

Install Driver for "Belkin F8T016":

Initially detected by ubuntu but doesn't work straight away. Needs two edits:

    sudo nano /etc/modprobe.d/blacklist
        blacklist hci_usb  < assert line
    sudo nano /etc/modules
        hci_usb reset=1  < assert line

The first edit stops ubuntu automatically loading the module and the second loads the module with the correct parameter.

sudo apt-get install pulseaudio-module-bluetooth
killall pulseaudio
if error erises:
Connection Failed: blueman.bluez.errors.DBusFailedError: Protocol not available.

************** 

After installing Debian with XFCE:
sudo apt install pulseaudio-module-bluetooth 
pulseaudio -k
pulseaudio --start
sudo apt-get install bluez-tools
run: bluetoothctl
scan on
trust [DEVNAME]
pair [DEVNAME]
connect [DEVNAME]

#############

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
short specs:
lspci
00:02.0 VGA compatible controller: Intel Corporation 2nd Generation Core Processor Family Integrated Graphics Controller (rev 09)
Full specs:
sudo lspci -v

---- Find video card information:
install lshw
sudo lshw -c video

********



###### REMOVE PACKAGE COMPLETELY:
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

*************************************
Drawing software for debian:
Pinta doesnt work

*************************************
Completely remove app with it's deps
sudo apt-get purge --auto-remove <package_name>

*************************************

find against freezing SSHFS:
unmount the drive:
fusermount -u -z /media/name_of_drive

sshfs -odebug,sshfs_debug,loglevel=debug
sshfs User@Server:/ /media/Server -o debug,sshfs_debug,loglevel=debug port=10000,allow_other,default_permissions



Another "hotfix" that worked for me is
umount /mount/point -f
I sudo-ed it, don't know if that's necessary for all.
Also, like above, don't autocomplete path with TAB. This "fix" made all my hung apps and windows complete their task queues and work as normal.

sshfs <email address hidden>:/users/username /mount/dir -o idmap=user -o allow_other -o reconnect


Workaround:

If you go to a different TTY (ctrl+alt+F1) in stead of a Terminal Emulator, tab-complete will have no problem. You can
umount --force /mount/point
It will fail, but repeat a bunch of times and it will unmount.
Switch back you your X session and all your resources are released and functional.
---------
Solution:
* configure ssh with a ServerAliveInterval value. When there's no traffic to the ssh server, the ssh client will check if the server is still alive every once in a while. This will keepp traffic alive which means that whatever is in between will not terminate the connection. You will want this value to be small but not so small that it sends packagesevery second. For me, 120 (2 minutes) solves the problems but others may need to set it at 30. Add the following to /etc/ssh/ssh_config or to ~/.ssh/config file

ServerAliveInterval 120

* mount with the ServerAliveInterval option. Same as before but rather than configuring ssh, you pass this option when using sshfs. Basically, mount with the following command:
sshfs user@host:dir mountpoint -o ServerAliveInterval=120
* use the sshfs specific reconnect option (I haven't tested this option)
sshfs user@host:dir mountpoint -o reconnect
For more information see "man ssh_config" for all ssh options and "man sshfs" for the sshfs specific options.
I do believe that this is poor design of sshfs but in their opinion the problem is that the user doesn't have its ssh client configured properly.
----------
Whenever I accidentally touch an sshfs mounted directory which is unavailable, I have to run "killall sshfs" to get things working again. This often happens when I mount a directory on my home network and touch that directory while at work.

Right now my workaround is to use the "ServerAliveInterval" option without the "reconnect" option. This means that when the ssh connection is lost, sshfs exits (and unmounts the locally mounted directory). I then need to run sshfs again when I want to use the mount again (I could also use autofs, I assume).

In my opinion the main problem is that sshfs leaves the local directory in an invalid state while it is "trying to reconnect". No matter what happenes, it seems clear that at any given time the mounted directory should either be mounted or unmounted -- meaning available or unavailable. If the underlying ssh connection has been lost, it makes no sense to keep the directory mounted (and locking up system calls). It's like if you pull a usb disk out of your computer. The media is unavailable. It should be unmounted, no matter what. This is more or less the behaviour I get without the reconnect option.

I would suggest that the reconnect option should simply act as an "outer loop" around a non-reconnecting version of sshfs. The non-reconnecting version should close the ssh connection and unmount the local directory on disconnect. The reconnect loop should then both connect and mount when a connection to the remote host is available again.

Obviously it is also a problem that many applications don't properly handle IO errors, and some even have IO operations in their main loops. But sshfs, fuse, and ssh can't do much to fix this.

----
sudo umount -l /mnt/mysshfs" 

for debbuging purpose use sshfs with options:
-o debug
-o sshfs_debug

To share between laptops / mount network drives:
Use SMB/Samba or NFS or SSHFS


SMBv3 on the server, CIFSv2.1 on *NIX clients, native on Windows. Honestly, this combination is the only one that allows me to get >20MB/s on my NAS. NFSv3 gave me 6MB/s, NFSv4 18MB/s, AFS wasn't something I was interested in, and SMBv3 gives me 45-50MB/s.
I don't use transport encryption, but SMBv3 supports it, and as long as you don't enable anonymous browsing your share will be secured behind decent auth.
Honestly I think this ticks all the boxes. As long as you're using SMBv3 you can really just use the defaults for your server (I mean, change the name and such), so configuration for both the server and the client are super simple, plus speed, security, and regular filesystem access.

You have some pretty good ideas. Already listed. Nfs and sshfs are both really good minus sshfs is slow with a secure encryption level. And nfs is simple but pretty easy to sniff filenames and capture files when they go over the network. Here are a couple ideas for the more paranoid thinking.
Iscsi with an encrypted LVM partition. Single node usage.
Docker with a Kerberos server and another docker running a nfs v4 server. Encrypted and stored securely.


https://blog.ja-ke.tech/2019/08/27/nas-performance-sshfs-nfs-smb.html

Solution for hanging SSHFS:


Use -o reconnect,ServerAliveInterval=15,ServerAliveCountMax=3

The combination ServerAliveInterval=15,ServerAliveCountMax=3 causes the I/O errors to pop out after one minute of network outage. This is important but largely undocumented. If ServerAliveInterval option is left at default (so without the alive check), processes which experience I/O hang seem to sleep indefinitely, even after the sshfs gets reconnect'ed. I regard this a useless behavior.

In other words what happens on -o reconnect without assigning ServerAliveInterval is that any I/O will either succeed, or hang the application indefinitely if the ssh reconnects underneath. A typical application becomes entirely hung as a result. If you'd wish to allow I/O to return an error and resume the application, you need ServerAliveInterval=1 or greater.

The ServerAliveCountMax=3 is the default anyway, but I like to specify it for readability.

######## XFCE Shortcuts #########

CTRL ALT D = Hide all windows
ALT + F3 = Application finder [ALT + F2]
Windows + L = Logout window
ALT + F10 = MAXIMIZE
【Super+p】 【XF86Display】

##################################
How to get motherboard model?
sudo dmidecode -t 2ddd


#########  G I T  ########

To undo git add before a commit:
Run git reset <file> or git reset to unstage all changes.
In older versions of git, the commands were git reset HEAD <file> and git reset HEAD respectively. This was changed in Git 1.8.2

-------------
Push specific commit:
$ git push <remote name> <commit hash>:<remote branch name>

# Example:
$ git push origin 2dc2b7e393e6b712ef103eaac81050b9693395a4:master

################################################



