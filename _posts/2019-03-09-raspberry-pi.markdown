---
published: true
title: raspberry pi
layout: post
---

This tutorial will guide you through the setup of your Raspberry Pi 
in *headless* mode as a **Time Capsule** and **AirPlay audio receiver**. 
Afterwards we will use *Docker* to setup an [**etherpad**](https://etherpad.org) server with group 
and user level pad authentication as well as a [**Nextcloud**](https://nextcloud.com) server along with [**WebRTC**](https://webrtc.org) for secure audio/video call conferencing.

## Materials

- [Raspberry 1373331 Pi 3 Modell B+ Mainboard, 1 GB](https://www.amazon.de/Raspberry-1373331-Pi-Modell-Mainboard/dp/B07BDR5PDW/ref=sr_1_5?s=computers&ie=UTF8&qid=1552132158&sr=1-5&keywords=raspberry+pi+3+b%2B) (34.49 EUR)
- [Raspberry Pi 3 official power supply](https://www.amazon.de/dp/B01N5ME6EW/ref=twister_B07BTCSCQ1?_encoding=UTF8&psc=1) (14.90 EUR)
- [SanDisk Ultra 64GB microSDXC card](https://www.amazon.de/SanDisk-Ultra-microSDHC-Speicherkarte-Adapter/dp/B073SB2L3C/ref=pd_bxgy_147_2/261-7932163-3214767?_encoding=UTF8&pd_rd_i=B073S9SFK2&pd_rd_r=6f8be1de-4261-11e9-aaff-27d05f5ca95c&pd_rd_w=H5Hrn&pd_rd_wg=Ce586&pf_rd_p=1ee75a10-e7c2-423a-9362-8396fcd2b687&pf_rd_r=NVJK17P8Y5H4KXWP0WHV&refRID=NVJK17P8Y5H4KXWP0WHV&th=1) (15.00 EUR)
- [Western Digital 4TB external hard drive](https://www.amazon.de/Western-Digital-Elements-Festplatte-WDBWLG0040HBK-EESN/dp/B00JT8AJZ0/ref=sr_1_3?s=computers&ie=UTF8&qid=1552132818&sr=1-3&keywords=wd+externe+festplatte+4tb) (94.99 EUR)
- Optional: [Raspberry Pi 3 official case](https://www.amazon.de/offizielles-Gehäuse-Raspberry-Himbeer-weiß/dp/B01CCPKCSK/ref=pd_bxgy_147_3/261-7932163-3214767?_encoding=UTF8&pd_rd_i=B01CCPKCSK&pd_rd_r=6f8be1de-4261-11e9-aaff-27d05f5ca95c&pd_rd_w=H5Hrn&pd_rd_wg=Ce586&pf_rd_p=1ee75a10-e7c2-423a-9362-8396fcd2b687&pf_rd_r=NVJK17P8Y5H4KXWP0WHV&psc=1&refRID=NVJK17P8Y5H4KXWP0WHV) (7.99 EUR)

Total: 167.37 EUR

## Starting your Raspberry Pi for the first time

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
If you wish to connect to your raspberry pi you need to set this up now by editing the `/boot/wpa_supplicant.conf` file with your wifi and country 2 letter code (here DE for Germany):
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=DE

network={
	ssid="my_network_name"
	psk="my_network_password"
}
```
If you do not want store your password in plain text you can use `wpa_passphrase` utility to generate an hashed password:
```
wpa_passphrase my_network_name my_network_password
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
and add git, vim and tmux (because we love it):
```
sudo apt-get install -y tmux vim git
```
Congratulations! Your Raspberry Pi is set and ready to go!

## Setting up your Raspberry Pi as an Airplay audio receiver

```
ssh pi
sudo apt-get install -y shairport-sync
```
Since the analog output is in mono only you might want to increase the quality of the audio coming out of you pi by using HDMI or a peripherical as shown [here](https://www.matthewwegner.com/raspberry-pi-airplay/).

Set your raspberry pi sound higher as the standard 50% is quite low:
```
sudo amixer set 'PCM' 75%
```

## Setting up your Raspberry Pi as a Time Capsule

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
Now that you've mounted your external drive make sure to never pull the power cable or USB on your raspberry pi as this might render the external drive read-only. For unmounting the drive you can 
```
sudo umount /dev/sda2
```
and for shutting down your raspberry pi you can 
```
sudo shutdown -h now 
```
Install the required packages for mounting your pi over afp and emulate a Time Capsule
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

## Installing Docker

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

## Building and running the Nextcloud and etherpad Docker image

As we will be generating ssl certificates with a Certificate Authority you will need to make sure
 your raspberry pi is accessible to the world. For this you will need to make sure your router is 
**forwarding ports 80 (http) and 443 (https)** to your raspberry pi. If you wish to backup you laptop wherever you are over the internet you can also set port 548 to be forwarded to your raspberry pi and then connect to your Time Capsuble over afp://myhostname.com. For instructions on how to set port forwarding consult your router's manual. 

Also, as you might not have a fixed IP you might wanna create a public *Hostname* with a *Free Dynamic DNS*. A good place for this is [noip.com](https://www.noip.com) for which you can install the *Dynamic Update Client (DUC)* so that your web address continuously points to your router's external IP address. You will need 2 hostnames, 1 for Nextcloud, 1 for etherpad.

Installing the *Dynamic Update Client (DUC)*
```
su
cd /usr/local/src
wget http://www.no-ip.com/client/linux/noip-duc-linux.tar.gz&redir_token=8N73LFVgQSnqLuVOQf0FMQHM58F8MTU1MjMzNDQ2MkAxNTUyMjQ4MDYy
tar xzf noip-duc-linux.tar.gz
cd noip-2.1.9-1/ && make && make install
/usr/local/bin/noip2
exit
```
To test that this is working you can have your router forward port 22 to your raspberry pi 
and try to ssh to your pi over the *Hostname* you created at noip.com. For safety reasons you might stop this forwarding once your test comes through.

Clone the repo:
```
cd ~/
git clone https://github.com/jorgeboucas/docker_nextcloud_etherpad.git
```
Edit the variables in the `Dockerfile` between `##### USER GIVEN VARIABLES #####` and `##### END OF USER GIVEN VARIABLES #####`. You might wanna check this [blog post](https://www.rosehosting.com/blog/generate-password-linux-command-line/) for help on choosing passwords.
```
##### USER GIVEN VARIABLES #####

# root password
ENV ROOTPASS=my_root_pass

# password for mysql
ENV myqsl_password=sqpass

# user and password for the database used by nextcloud
ENV MAINDB=nextcloud
ENV PASSWDDB=ncdbpass

# user and password for the nextcloud administrator
ENV ADMIN=nextcloud
ENV ADMINPASS=ncpass

# server info for ssl certificates
ENV SERVER_ADDRESS=nextcloud.myhostname.com
ENV SERVER_ADMIN="my.email@email.com"

# company info for hacking the nextcloud's landing page
ENV COMPANY_NAME=company_name
ENV COMPANY_WEBPAGE='http:\/\/www.company.com'
ENV COMPANY_SLOGAN='My Company Slogan'

# internal host ip for trusted hosts
ENV HOST_INTERNAL_IP=pi_ip_on_my_network

# user and password for the database used by etherpad
ENV ETHERPAD_MYSQL_USER=ETHERPAD_MYSQL_USER
ENV ETHERPAD_MYSQL_PASS=ETHERPAD_MYSQL_PASS

# user and password for the etherpad administrator
ENV ETHERPAD_ADMIN=ETHERPAD_ADMIN
ENV ETHERPAD_ADMIN_PASS=ETHERPAD_ADMIN_PASS

# etherpad address, ip bind,  and port
ENV ETHERPAD_WEB_ADDRESS=etherpad.myhostname.com

##### END OF USER GIVEN VARIABLES #####
```
Build the image and run the container:
```
cd ~/docker_nextcloud_etherpad
docker built -t webserver .
```
Run the container mapping the certificates `/etc/letsencrypt`, nextcloud `/var/www/html`, 
databases `var/lib/mysql`, and etherpad `/etherpad-lite` folders to your external hard drive. If you wish to create backups of your Nextcloud and etherpad instances these are the folders you need to copy/backup to a separate/3rd drive. If you ever restart your raspberry pi you can relaunch your instances by starting here:
```
cd /media/time_machine
mkdir -p persistent_data/letsencrypt \
persistent_data/nextcloud \
persistent_data/db \
persistent_data/etherpad && \
docker run \
-v $(pwd)/persistent_data/letsencrypt:/etc/letsencrypt \
-v $(pwd)/persistent_data/nextcloud:/var/www/html \
-v $(pwd)/persistent_data/db:/var/lib/mysql \
-v $(pwd)/persistent_data/etherpad:/etherpad-lite \
--name cloud -d -p 80:80 -p 443:443 -p 9001:9001 webserver /bin/bash
```
If you wish to get a shell on the running container you can do so by 
```
docker exec -it cloud /bin/bash
```
You can now access your Nextcloud instance at your defined hostname `nextcloud.myhostname.com` and etherpad at your defined `etherpad.myhostname.com`. 

For login in to Nextcloud you can use the values you've set for `ADMIN` and  `ADMINPASS`. You can add users to yout Nextcloud instace by using the managment console inside Nexcloud. 

For *WebRTC* you can use the *Talk* app inside Nextcloud.

For login to etherpad you can use the values you've set to `ETHERPAD_ADMIN` and `ETHERPAD_ADMIN_PASS`. For private pads you can access `etherpad.myhostname.com/mypads` which you 
can administrate on `etherpad.myhostname.com/mypads/?/admin`.

<p style="text-align: center;"><strong><a target="_blank" href="https://github.com/jorgeboucas/jorgeboucas.github.io/issues">Issues/comments welcome!</a></strong></p>