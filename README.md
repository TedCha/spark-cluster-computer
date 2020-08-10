# raspberry-pi-4-spark-cluster
Project that details the creation of a Spark Cluster using Raspberry Pi 4 and Ubuntu Server LTS 20.04

## Introduction

## Physical Cluster Setup
#### 1. Server Rack

To start the setup, install your Raspberry Pis on the server rack casing. You can find these cases for cheap on Amazon.

<p float="left">
  <img src="./assets/images/physical_2.jpg" width="250" />
  <img src="./assets/images/physical_3.jpg" width="250" /> 
  <img src="./assets/images/physical_4.jpg" width="250" />
</p>

#### 2. Network Connection

For my setup, I used a wired ethernet connection from my router which was then routed to my Netgear 4-port 100 Mbps Switch. Then, I connected each Raspberry Pi to the switch via a 1-foot ethernet cable.

<p float="left">
  <img src="./assets/images/physical_5.jpg" width="250" />
  <img src="./assets/images/physical_7.jpg" width="250" /> 
  <img src="./assets/images/physical_8.jpg" width="250" />
</p>

#### 3. Power Supply

I used a ![RAVPower 4-Port USB Power Supply](https://www.amazon.com/Charging-Station-RAVPower-Charger-Compatible/dp/B00OT6YUIY/ref=sr_1_4?dchild=1&keywords=ravpower+4+port&qid=1597074773&sr=8-4) to power my Raspberry Pi. As they are Raspberry Pi 4s, they all use USB-C as a power supply.

<p float="left">
  <img src="./assets/images/physical_9.jpg" width="250" />
  <img src="./assets/images/physical_10.jpg" width="250" /> 
  <img src="./assets/images/physical_11.jpg" width="250" />
</p>

## Individual Raspberry Pi Setup

#### 1. Ubuntu Server LTS 20.04 Installation

Use the ![Raspberry Pi Imager](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#2-prepare-the-sd-card) to write the Ubuntu Server LTS 20.04 64 Bit to each pi.

<p float="left">
  <img src="https://ubuntucommunity.s3.dualstack.us-east-2.amazonaws.com/optimized/2X/3/3bd138fa7bcddc12f303c9c26a004601ace296b8_2_690x439.png"/>
</p>

#### 2. Pi Configuration

SSH into each Pi individually and setup some basic configuration. Use this net-tools command to find the IP address of each Pi.

```bash
arp -na | grep -i "dc:a6:32"
```

If that doesn't work, you can also use your router admin interface to see a list of connected devices.

After you're connected you'll be prompted to change the password. Change it to the same password on each Pi; something secure but easy to remember. 

Ensure that each Pi has their time synchronized using the following command:

```bash
timedatectl status
```

You should be returned this prompt:

```bash
               Local time: Mon 2020-08-10 12:24:54 EDT  
           Universal time: Mon 2020-08-10 16:24:54 UTC  
                 RTC time: Mon 2020-08-10 16:24:54      
                Time zone: America/New_York (EDT, -0400)
System clock synchronized: yes                          
              NTP service: active                       
          RTC in local TZ: no          
```

If the system clock is synchronized and NTP service is active on each Pi, you're good to go. 

Lastly, run the following commands on each Pi to finish individual configuration:

```bash
sudo apt update
```

```bash
sudo apt upgrade
```

```bash
sudo reboot
```
## Cluster Setup

## Hadoop Installation

## Spark Installation
