---
published: true
title: raspberry pi
layout: post
---

This tutorial will guide you through the setup of your Raspberry Pi 
in *headless* mode as a **TimeCapsule** and **AirPlay audio receiver**. 
Afterwards we will use *Docker* to setup an [**etherpad server**](https://etherpad.org) with group 
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
to your microsd card.

On your Mac, remove the microcard and insert it back in again. For making 
sure you will be able to connect over ssh to you raspberry pi you will need to
```cd /Volumes/boot
touch ssh```
As we will use the raspberry pi without a screen we decrease the amount of memmory available 
for graphics
```cd /Volumes/boot
echo gpu_mem=16 >> config.txt```

