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

```
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

The following steps will need to be done on each Pi.

#### 1. Static IP Setup

Ubuntu Server LTS 20.04 requires Netplan for network configuration. Specifically, editing a few yaml files.

First, find the name of your network interface name by running:

```bash
ip a
```

The returned information should look like so:
<pre>
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: <b>eth0</b>: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 7c:97:0d:a6:27:53 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.1/24 brd 192.168.0.255 scope global dynamic noprefixroute en
...
</pre>

The network interface name is the bolded `eth0` tag. We'll need this later.

Next, you'll need to use nano or vim to edit the configuration files. In this tutorial, I'll be using nano. Use the following commands to edit the configuration file to disable automatic network configuration.

```bash
sudo nano /etc/cloud/cloud.cfg.d/99-disable-config.cfg
```

All you need to add is the following code:

```cfg
network: {config: disabled}
```

Then, you'll setup the static IP by editing the 50-cloud-init.yaml file. Use the following command:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

My configuration file looked like so:
(X being the last digit of the specific IP address for each Pi; 10.1.2.121, 10.1.2.122, 10.1.2.123, etc.)

```yaml
network:
    ethernets:
        eth0:
            dhcp4: false
            addresses: [10.1.2.12X/24]
            gateway4: 10.1.2.1
            nameservers:
                addresses: [10.1.2.1, 8.8.8.8]
    version: 2
```

The basic template is:
```yaml
network:
    ethernets:
        {Network Interface Name}:
            dhcp4: false
            addresses: [{Specifc IP Adress}/24]
            gateway4: {Gateway Address}
            nameservers:
                addresses: [{Gateway Address}, 8.8.8.8]
    version: 2
```

After edit the file, apply the settings by using the following commands:

```bash
sudo netplan try
```

```bash
sudo netplan apply
```

Then reboot the Pi and confirm the static IP address is set correctly.

#### 2. Hosts/Hostname Configuration

On each Pi, you'll have to edit the hosts and hostnames files to the specific Pi information. First we'll update the hostname file using the following command:

```bash
sudo nano /etc/hostname
```

The hostname file should like so (X being the last digit of the specific IP address for each Pi):

```
pi0X
```

For example, the hostname file for pi01 will look like:

```
pi01
```

Next, we'll have to update the hosts file using the following command:

```bash
sudo nano /etc/hosts
```

The hosts file should look like so after editing:

```
# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

10.1.2.121 pi01
10.1.2.122 pi02
10.1.2.123 pi03
```

While editing the hosts file, make sure to delete the localhost 127.0.0.1 instance from the file. The template for adding additional nodes to the hosts file is:

```
{IP Address} {hostname}
```

Now reboot the Pi and repeat the steps on the rest of the nodes. Note, the hostname file will be different for each node, but the hosts file should be exactly the same.

#### 3. Public Key SSH Authentication Configuration

First, edit the ssh config file on the Pi using the following command:

```bash
nano ~/.ssh/config
```

Your config file should look like so: 

```
Host pi01
User ubuntu
Hostname 10.1.2.121

Host pi02
User ubuntu
Hostname 10.1.2.122

Host pi03
User ubuntu
Hostname 10.1.2.123
```

The template for adding more nodes to the config file is:

```
Host pi0X
User ubuntu
Hostname {IP Address}
```

Next, create an SSH key pair on the Pi using:

```bash
ssh-keygen -t rsa -b 4096
```

You'll be prompted a few times. Press enter through them rather than change anything because the key pair needs to exist in the .ssh directory and the key pair should be passwordless.

The output should look similar to this:

```
Your identification has been saved in /your_home/.ssh/id_rsa
Your public key has been saved in /your_home/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:/hk7MJ5n5aiqdfTVUZr+2Qt+qCiS7BIm5Iv0dxrc3ks user@host
The key's randomart image is:
+---[RSA 4096]----+
|                .|
|               + |
|              +  |
| .           o . |
|o       S   . o  |
| + o. .oo. ..  .o|
|o = oooooEo+ ...o|
|.. o *o+=.*+o....|
|    =+=ooB=o.... |
+----[SHA256]-----+
```

Now, repeat the SSH keygen on Pi 2 and 3. 

Then use the following command to copy the public keys from Pi 2 and 3 to Pi 1.

```bash
ssh-copy-id pi01@10.1.2.121
```

If your IP address/hostname is different than mine, the template is:

```bash
ssh-copy-id {hostname}@{IP Address}
```

Then SSH back into Pi 1 and use the following command to copy Pi 1's public key into it's authorized keys:

```bash
ssh-copy-id pi01
```

Finally, you can copy Pi 1's configuration files to the rest of the Pi's using the following commands:

```bash
scp ~/.ssh/authorized_keys pi0X:~/.ssh/authorized_keys
```

```bash
scp ~/.ssh/config pi0X:~/.ssh/config
```

The last step is disable Password Authentication in the ssh config files using the following command:

```bash
sudo nano /etc/ssh/sshd_config
```

In the file, you'll find an configuration called `PasswordAuthentication`. If it's commented out, uncomment it and set the value to no. It should look like this:

```
...
PasswordAuthentication no
...
```

Now you should be able to ssh into any Pi from any of the Pis without providing a password.


#### 4. Cluster Ease of Use Functions

This step entails editing the `.bashrc` file in order to create some custom functions for ease of use. 

On the master Pi, we'll first edit the ~/.bashrc file:

```bash
nano ~/.bashrc
```

Within this file, add the following code to the bottom of the file:

```bash
# cluster management functions

#   list what other nodes are in the cluster
function cluster-other-nodes {
    grep "pi" /etc/hosts | awk '{print $2}' | grep -v $(hostname)
}

#   execute a command on all nodes in the cluster
function cluster-cmd {
    for node in $(cluster-other-nodes);
    do
        echo $node;
        ssh $node "$@";
    done
    cat /etc/hostname; $@
}

#   reboot all nodes in the cluster
function cluster-reboot {
    cluster-cmd sudo reboot now
}

#   shutdown all nodes in the cluster
function cluster-shutdown {
    cluster-cmd sudo shutdown now
}

function cluster-scp {
    for node in $(cluster-other-nodes);
    do
        echo "${node} copying...";
        cat $1 | ssh $node "sudo tee $1" > /dev/null 2>&1;
    done
    echo 'all files copied successfully'
}

#   start yarn and dfs on cluster
function cluster-start-hadoop {
    start-dfs.sh; start-yarn.sh
}

#   stop yarn and dfs on cluster
function cluster-stop-hadoop {
    stop-dfs.sh; stop-yarn.sh
}
```

Now use the following command to copy the .bashrc to all the worker nodes:

```bash
cluster-scp ~/.bashrc
```

Lastly, run the following command to source the .bashrc file on all nodes:

```bash
cluster-cmd source ~/.bashrc
```

## Hadoop Installation

## Spark Installation
