# sd-images
images and instructions for loading Ubuntu onto a sd card, for use in Jetson Nanos and Raspberry Pis


## Flashing a new SD card for Ubuntu server on Raspi 4b

On a host computer
1. Download and install the Raspi Imager (https://www.raspberrypi.org/software/) on a host machine
2. Open raspi imager, and choose OS > Other General Purpose > Ubuntu > Ubuntu 20.04.2 LTS 64 bit
3. Plug in the sd card and choose it in raspi imager
4. Click on Write (this step takes a while)
5. Once done, remove the sd and plug it into a raspi

Since the raspi is configured as a ubuntu server, it does not contain a desktop by default. Neither is it connected to wifi.

When booted with a screen connected (hdmi cable must be connected before powering up the raspi)
after the boot sequence, you will be prompted to a command line interface, and see something like:

```
ubuntu login: 
```
and possibly more text - you can usually ignore this additional text. 

To log in, use the username `ubuntu` and password `ubuntu`. You will need to change the passsword on the first boot. 

## Adding internet
1. 
```
cd /etc/netplan/
```
2. edit a file, which contains the network information
```
sudo vim 50-cloud-init.yaml
```
vim or nano, based on what you are used to. 
3. Add in an interface to connect to a wifi network

my final file looks like (when with a password)
```
# a bunch of comments
network:
    ethernets:
        eth0:
            dhcp4: true
            optional: true
    version: 2
    # add wifi setup information here ...
    wifis:
        wlan0:
            optional: true
            access-points:
                "YOUR-SSID-NAME":
                    password: "YOUR-NETWORK-PASSWORD"
            dhcp4: true
```
or when the wifi network is not protected by a password:
```
# a bunch of comments
network:
    ethernets:
        eth0:
            dhcp4: true
            optional: true
    version: 2
    # add wifi setup information here ...
    wifis:
        wlan0:
            optional: true
            access-points:
                "YOUR-SSID-NAME": {}
            dhcp4: true
```
be careful about indentation - it should be using 4 spaces for each indentation

now run 
```
sudo netplan generate
sudo netplan apply
```
this should enable wifi on the raspi. you can test it by doing
```
ping google.com
```

## Adding SSH access
1. Determine the ip address of the raspi: `ip a` which has a field `wlan0` : `inet` and gives you the ip address
2. Check if ssh access is working. On a different computer access it by typing ```ssh ubuntu@<ip address found above>``` and use the password of the computer
3. If this didnt work, you might need to install `openssh-server` by doing `sudo apt install openssh-server`

## Update, upgrade and reboot
1.
```
sudo reboot
```
2. 
```
sudo apt-get update && sudo apt-get upgrade
```
dont forget to hit yes before you go for lunch.
3. 
```
sudo reboot
```

## Adding a Lightweight Desktop Environment
Follow these stepos:


## Install ROS





