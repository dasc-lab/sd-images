# Flashing a new SD card for Ubuntu server 20.04 on Raspi 4B

## SIMPLE METHOD (~1 hour)
1. go to the lab dropbox folder, and download the sd card image `raspi4-ubuntu20.img` (currently available at https://www.dropbox.com/s/q1g2etbq7a4hugb/raspi4_ubuntu20.img.gz?dl=0)
2. unzip the .gz file:
```
gunzip raspi4-ubuntu20.img
```
this will create a file about 32 gb big, and needs to be loaded onto a SD card at least 32 gb large.

3. Open `Raspi-Imager` (https://www.raspberrypi.org/software/)
4. Choose `Custom Image` and select the `raspi4-ubuntu20.img` file (technically you might be able to use the compressed .gz file directly, ie, skipping step 2, but it didnt work for me last time.
5. choose the sd card, and click ok to upload it.

Theoretically, this should create a new sd card that is identical to the one that is achieved by following the steps below.


## FULL STEPS (~3 hours)

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

## Disable the uBOOT
When the raspi is turned on, it first goes into a `uBOOT` mode, where it is waiting to check if the user sends any keyboard commands. if the user does, it goes into a safe mode, allowing you to make modifications. However, for our applications, we will be connecting the pixhawk using UART connections. Unfortunately, these messages also go to the same place, and therefore the bootloader gets interrupted at boot. We need to disable this. The description here is a copy of the steps listed at https://stackoverflow.com/questions/34356844/how-to-disable-serial-consolenon-kernel-in-u-boot/64583919#64583919

**OPTION 1: Prebuilt**
1. Backup the current raspi uboot file 
```
cd /boot/firmware
mv uboot_rpi_4.bin uboot_rpi_4.bin.bak
```
2. Copy the `uboot_rpi_4.bin` from this repo into the same directory.
3. `sudo reboot`

**OPTION 2: From Source**
1. `sudo apt install git make gcc gcc-aarch64-linux-gnu bison flex -y`
2. `git clone --depth 1 git://git.denx.de/u-boot.git`
3. `cd u-boot`
4. Find your board config files, inside the `configs` folder - for this set up, we modified `rpi_4_defconfig` Add the following lines to the end of the file"
```
CONFIG_BOOTDELAY=-2
CONFIG_SILENT_CONSOLE=y
CONFIG_SYS_DEVICE_NULLDEV=y
CONFIG_SILENT_CONSOLE_UPDATE_ON_SET=y
CONFIG_SILENT_U_BOOT_ONLY=y
```
The first line removes the boot delay, so autoboot will not be interrupted by messages sent on UART interface. Next four lines enable silent boot, so U-boot will not send any messages on UART itself, because the messages might in turn confuse your device. 

5. One more little thing left, set silent boot environmental variable. Change the header file for your board (for raspberry pi it is `include/configs/rpi.h` ). Search for `CONFIG_EXTRA_ENV_SETTINGS` and add the `"silent... ` line so that the it looks like
```
#define CONFIG_EXTRA_ENV_SETTINGS \
    "dhcpuboot=usb start; dhcp u-boot.uimg; bootm\0" \
    "silent=1\0" \
    ENV_DEVICE_SETTINGS \
    ENV_DFU_SETTINGS \
    ENV_MEM_LAYOUT_SETTINGS \
    BOOTENV
```

6. `cd` back into the u-boot folder
7. and run `make rpi_4_defconfig`
8. and run `make CROSS_COMPILE=aarch64-linux-gnu-` 
9. When build process finishes you will have a `u-boot.bin` file. 
10. Rename this file to `uboot_rpi_4.bin`, and move it to the `/boot/firmware` folder. You should probably also create a backup of the original `uboot` file:
```
cd /boot/firmware/
sudo mv uboot_rpi_4.bin uboot_rpi_4.bin.bak
```
Now copy in the new file from the github repo to the boot folder
```
sudo cp <location of u-boot repo>/u-boot.bin /boot/firmware/uboot_rpi_4.bin
```

11. `sudo reboot` or `sudo poweroff`

The raspi should still reboot correctly


## Disable Bluetooth/Enable Serial 0/Change udev rules
References: JPL: https://github.com/nasa-jpl/osr-rover-code/blob/foxy-devel/setup/rpi.md#5-setting-up-serial-communication-on-the-rpi

References: Summary: https://askubuntu.com/questions/1254376/enable-uart-communication-on-pi4-ubuntu-20-04

1. Disable serial-getty@ttyS0.service
```
sudo systemctl stop serial-getty@ttyS0.service
sudo systemctl disable serial-getty@ttyS0.service
sudo systemctl mask serial-getty@ttyS0.service
```
2. Setup udev rules: put below content in *new* file `/etc/udev/rules.d/10-local.rules`
```
KERNEL=="ttyS0", SYMLINK+="serial0" GROUP="tty" MODE="0660"
KERNEL=="ttyAMA0", SYMLINK+="serial1" GROUP="tty" MODE="0660"
```
Reload udev rules: `sudo udevadm control --reload-rules && sudo udevadm trigger`

3. Add user to tty group: `sudo adduser ubuntu tty`
4. Delete substring `console=serial0,115200` from `/boot/firmware/cmdline.txt`
5. (DONT DO THIS STEP) Add newline `dtoverlay=disable-bt` to `/boot/firmware/config.txt` (I put it right under the `cmdline=cmdline.txt` line)
6. Change `/boot/firmware/usercfg.txt` and include the line `dtoverlay=pi3-disable-bt` (note, its pi3 not pi4!)
7. Restart
8. If you `ls /dev/` you should now see `serial0` and `serial1`


Btw `raspi-config` didnt work for us. Even after we force installed it

## Change the name of the computer (optional)
1. `sudo vim /etc/hostname`
2. change the word `ubuntu` to the desired name of the computer. The username is still `ubuntu` and when ssh-ing in, you still use `ubuntu@<ip address>`

## Adding a Lightweight Desktop Environment (Optional)
1. sudo apt-get update && sudo apt-get upgrade
2. `sudo apt-get install tasksel`
3. `sudo apt-get install slim`
4. `sudo tasksel install ubuntu-mate-core` (this step takes a while)
5. `sudo reboot`
6. log in using the correct username and password

To disable or enable the gui, press one of `Ctrl + Alt + F1` or `Ctrl + Alt + F2` or `Ctrl + Alt + F3` . I think the first one worked for me.


## Install ROS NOETIC
Run all of these lines (seems like its better to run them one at a time?)
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```
```
sudo apt install curl -y
```
```
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
```
```
sudo apt update
```
```
sudo apt install ros-noetic-desktop -y
```
```
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
```
```
source ~/.bashrc
```
```
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential python3-roslaunch -y
```
(this step take some time)

```
sudo apt install python3-rosdep2 -y
```
```
sudo rosdep init
```
```
rosdep update
```

```
sudo apt install ros-noetic-rosbash
```

## Install MAVROS

Install mavros and other dependencies
```
sudo apt-get install ros-noetic-mavros* -y
wget https://raw.githubusercontent.com/mavlink/mavros/master/mavros/scripts/install_geographiclib_datasets.sh
sudo bash ./install_geographiclib_datasets.sh
```


## Testing
1. Connect the pixhawk using UART cables
2. power on the raspi
3. `roslaunch mavros px4.launch fcu_url:=/dev/serial0:921600` and it shouldnt show any new issues!


## Saving
The sd card image at the end of these steps were saved by running the commands
```
sudo dd if=/dev/sdd of=~/Devansh/raspi4_backup.img bs=4M status=progress
```
on a different computer when the sd card was plugged in

This `.img` file is uploaded in this repo, and can be cloned for a direct copy in the future

Potentially faster if you also compress it as you clone it(?):
```
sudo dd bs=10M status=progress if=/dev/sdd | gzip -c --fast > ~/Devansh/raspi4_backup.img 
```
