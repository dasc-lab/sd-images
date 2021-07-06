## Flashing a new SD card for Ubuntu server 20.04 on Raspi 4B

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
1. `cd /etc/netplan/`
2. edit the file which contains network info: `sudo nano 50-cloud-init.yaml`
3. Add in an interface to connect to a wifi network

My final file looks like (when with the network has a password)
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
Note, this also means that you can no longer change the network from a gui. If you want to change the wifi the raspi is connected to, change the netplan file appropriately.

## Adding SSH access
1. Determine the ip address of the raspi: `ip a` which has a field `wlan0` : `inet` and gives you the ip address
2. Check if ssh access is working. On a different computer access it by typing ```ssh ubuntu@<ip address found above>``` and use the password of the computer
3. If this didnt work, you might need to install `openssh-server` by doing `sudo apt install openssh-server`

## Update, upgrade and reboot
1. `sudo reboot`
2. `sudo apt-get update && sudo apt-get upgrade`
dont forget to hit yes before you go for lunch.
3. `sudo reboot`

## Change the name of the computer
1. `sudo vim /etc/hostname`
2. change the word `ubuntu` to the desired name of the computer. The username is still `ubuntu` and when ssh-ing in, you still use `ubuntu@<ip address>`

## Disable the uBOOT
When the raspi is turned on, it 

## Adding a Lightweight Desktop Environment (Optional)
1. sudo apt-get update && sudo apt-get upgrade
2. `sudo apt-get install tasksel`
3. `sudo apt-get install slim`
4. `sudo tasksel install ubuntu-mate-core` (this step takes a while)
5. `sudo reboot`
6. log in using the correct username and password


## Install ROS NOETIC
Run all of these lines (just paste all the lines into a terminal)
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt install curl -y
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
sudo apt update
sudo apt install ros-noetic-desktop -y
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential -y
```
(this step take some time)

```
sudo rosdep init
rosdep update
```



