---
published: true
title: raspberry pi
layout: post
---

This tutorial will guide you through the setup of your Raspberry Pi 
in *headless* mode as a **Time Capsule** and **AirPlay audio receiver**. 
Afterwards we will use *Docker* to setup an [**etherpad**](https://etherpad.org) server with group 
and user level pad authentication as well as a [**Nextcloud**](https://nextcloud.com) server along with [**WebRTC**](https://webrtc.org) for secure audio/video call conferencing.

### Materials

- [Raspberry 1373331 Pi 3 Modell B+ Mainboard, 1 GB](https://www.amazon.de/Raspberry-1373331-Pi-Modell-Mainboard/dp/B07BDR5PDW/ref=sr_1_5?s=computers&ie=UTF8&qid=1552132158&sr=1-5&keywords=raspberry+pi+3+b%2B) (34.49 EUR)
- [Raspberry Pi 3 official power supply](https://www.amazon.de/dp/B01N5ME6EW/ref=twister_B07BTCSCQ1?_encoding=UTF8&psc=1) (14.90 EUR)
- [SanDisk Ultra 64GB microSDXC card](https://www.amazon.de/SanDisk-Ultra-microSDHC-Speicherkarte-Adapter/dp/B073SB2L3C/ref=pd_bxgy_147_2/261-7932163-3214767?_encoding=UTF8&pd_rd_i=B073S9SFK2&pd_rd_r=6f8be1de-4261-11e9-aaff-27d05f5ca95c&pd_rd_w=H5Hrn&pd_rd_wg=Ce586&pf_rd_p=1ee75a10-e7c2-423a-9362-8396fcd2b687&pf_rd_r=NVJK17P8Y5H4KXWP0WHV&refRID=NVJK17P8Y5H4KXWP0WHV&th=1) (15.00 EUR)
- [Western Digital 4TB external hard drive](https://www.amazon.de/Western-Digital-Elements-Festplatte-WDBWLG0040HBK-EESN/dp/B00JT8AJZ0/ref=sr_1_3?s=computers&ie=UTF8&qid=1552132818&sr=1-3&keywords=wd+externe+festplatte+4tb) (94.99 EUR)
- Optional: [Raspberry Pi 3 official case](https://www.amazon.de/offizielles-Gehäuse-Raspberry-Himbeer-weiß/dp/B01CCPKCSK/ref=pd_bxgy_147_3/261-7932163-3214767?_encoding=UTF8&pd_rd_i=B01CCPKCSK&pd_rd_r=6f8be1de-4261-11e9-aaff-27d05f5ca95c&pd_rd_w=H5Hrn&pd_rd_wg=Ce586&pf_rd_p=1ee75a10-e7c2-423a-9362-8396fcd2b687&pf_rd_r=NVJK17P8Y5H4KXWP0WHV&psc=1&refRID=NVJK17P8Y5H4KXWP0WHV) (7.99 EUR)

Total: 167.37 EUR

### Starting your Raspberry Pi for the first time

Download the *"Raspbian Stretch Lite"* image from the Raspbian downloads 
page [here](https://www.raspberrypi.org/downloads/raspbian/) and follow 
raspberrypi.org [instructions](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) 
on how to burn the image.

Shortly, after downloading the [zip file](https://downloads.raspberrypi.org/raspbian_lite_latest) 
use [balenaEtcher](https://www.balena.io/etcher/) to burn the Raspbian image 
to your micro sd card.

On your Mac, remove the micro sd card and insert it back in again. For making 
sure you will be able to connect over ssh to you raspberry pi you will need to
```
cd /Volumes/boot
touch ssh
```
As we will use the raspberry pi without a screen we decrease the amount of memmory available 
for graphics
```
cd /Volumes/boot
echo gpu_mem=16 >> config.txt
```
Insert the micro sd card into your raspberry pi. Connect the ethernet cable and power cable to start 
your raspberry pi. From your Mac connect to the pi on your network by
```
ssh pi@raspberrypi
```
The original password is `raspberry`. 

Setup your user by replacing `MYUSERNAME` with your username and `MYPASSWORD` with your own password:
```
sudo -s
echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && locale-gen
useradd -m -s /bin/bash -N -u 1001 MYUSERNAME
echo "root:MYPASSWORD" | chpasswd
echo "pi:MYPASSWORD" | chpasswd
echo "MYUSERNAME:MYPASSWORD" | chpasswd
adduser MYUSERNAME sudo
exit
exit
```
If you don't yet have SSH keys on your Mac follow the instructions [here](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html) to generate them. Copy your ssh keys to your pi 
```
ssh-copy-id MYUSERNAME@raspberrypi
```
Using the password you just used to replace `MYPASSWORD` above. Edit your Mac's `.ssh/config` file 
```
vim ~/.ssh/config
```
to include
```
Host pi
    HostName raspberrypi
    Port 22
    User MYUSERNAME
```
You can now connect password free to your pi with:
```
ssh pi
```
and remove the original `pi` user:
```
sudo userdel -r pi
```
and add tmux (because we love it):
```
sudo apt-get install -y tmux
```
Congratulations! Your Raspberry Pi is set and ready to go!

### Setting up your Raspberry Pi as an Airplay audio receiver

```
ssh pi
sudo apt-get install -y shairport-sync
```
As shown [here](https://www.matthewwegner.com/raspberry-pi-airplay/) you can also increase the quality of the audio coming out of you pi by using a peripherical.

### Setting up you Raspberry Pi as a Time Capsule

Connect an empty HFS+ formated external drive to you pi over USB.

Install required packages for dealing with HFS in linux:
```
sudo apt-get install -y hfsprogs hfsplus
```
Find your disk and device name (usually `/dev/sda2`):
```
sudo fdisk -l
```
Format the drive
```
sudo mkfs.hfsplus -v my_external_drive /dev/sda2
```
Create the mounting point, add it to `/etc/fstab`, mount the device, and make sure you own the mount point:
```
su
mkdir -p /media/time_machine
echo "/dev/sda2 /media/time_machine hfsplus force,rw,user,auto 0 0" >> /etc/fstab
mount -a
chown -R MYUSERNAME /media/time_machine
```
Install the required packages for mounting you pi over afp and emulate a Time Capsule
```
su
aptitude install -y build-essential libevent-dev libssl-dev \
libgcrypt11-dev libkrb5-dev libpam0g-dev libwrap0-dev \
libdb-dev libtdb-dev avahi-daemon libavahi-client-dev \
libacl1-dev libldap2-dev libcrack2-dev systemtap-sdt-dev \
libdbus-1-dev libdbus-glib-1-dev libglib2.0-dev \
libio-socket-inet6-perl tracker libtracker-sparql-1.0-dev \
libtracker-miner-1.0-dev libmariadbclient-dev

wget http://prdownloads.sourceforge.net/netatalk/netatalk-3.1.11.tar.gz
tar -xf netatalk-3.1.11.tar.gz
cd netatalk-3.1.11/
./configure \
        --with-init-style=debian-systemd \
        --without-libevent \
        --without-tdb \
        --with-cracklib \
        --enable-krbV-uam \
        --with-pam-confdir=/etc/pam.d \
        --with-dbus-daemon=/usr/bin/dbus-daemon \
        --with-dbus-sysconf-dir=/etc/dbus-1/system.d \
        --with-tracker-pkgconfig-version=1.0
make 
make install
netatalk -V
echo """[Global]
  mimic model = TimeCapsule6,106
[Time Machine]
  path = /media/time_machine
  time machine = yes""" > /usr/local/etc/afp.conf
systemctl enable avahi-daemon
systemctl enable netatalk
service avahi-daemon start
service netatalk start
```
You Mac should now be seeing `raspberrypi` on your network as a Time Capsule and you should be able
 to mount it by clicking on it on the *Finder* and select it as a Time Machine destination in 
 *System Preferences* > *Time Machine* > *Add or Remove Backup Disk...*.

### Installing Docker

[Docker](https://www.docker.com) is great! 
```
sudo apt-get update
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $USER
sudo gpasswd -a $USER docker
docker --version
sudo reboot
```
Login to your pi again once it has rebooted ```ssh pi``` 
```
docker login
sudo docker engine activate
```
and test your docker installation:
```
docker run armhf/hello-world
```
Here the message you should see
```
Hello from Docker on armhf!
This message shows that your installation appears to be working correctly.
```

### Building and running the Nextcloud and etherpad Docker image

As we will be generating ssl certificates with a Certificate Authority you will need to make sure
 your raspberry pi is accessible to the world. For this you will need to make sure your router is 
**forwarding ports 80 (http) and 443 (https)** to your raspberry pi. For instructions of how to do this consult your router's manual. 

Also, as you might not have a fixed IP you might wanna have as a starter a *Free Dynamic DNS*. A good place for this is [noip.com](https://www.noip.com) for which you can install the *Dynamic Update Client (DUC)* so that your web address continuously points to your router's external IP address.

A good way to test that this is working is to have your router forward port 22 to your raspberry pi 
and try to ssh to your pi over the *Hostname* you created at noip.com.
