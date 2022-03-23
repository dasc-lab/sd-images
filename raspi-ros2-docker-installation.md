# Installation of Ros 2 and Docker on Raspi 4

## STEP 1: Load Ubuntu
I followed this guide: https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview

1. Using RPI Imager, load Ubuntu onto the SD card. I chose Ubuntu 20.04 Server (64 bit, RPi 3/4/400). 
2. Set up wifi: Edit the `network-config` file on the sd card, and put in something like the following:
    ```
    network:
        version: 2
        wifis:
            wlan0:
                dhcp4: true
                optional: true
                access-points:
                <wifi network name>:
                    password: "<wifi password>"
    ```

    If there is no password:
    ```
        network:
            version: 2
            # add wifi setup information here ...
            wifis:
                wlan0:
                    optional: true
                    access-points:
                        "YOUR-SSID-NAME": {}
                    dhcp4: true
    ```

3. Plug the SD card into the raspi. Optionally, connect screen and keyboard and mouse. 
4. Boot the raspi. You might need to reboot it once to get it to connect to the internet/be able to ssh into it.
5. The default user and password are `ubuntu` and `ubuntu`. At first login, it will ask you to change the password. 
6. Change the hostname: `sudo vim /etc/hostname`, and change it `rpi9` or any unique ID you want. Now, to access the raspi, `ssh ubuntu@rpi9.local` can be used.
7. [Optional] Disable auto-updates:  `sudo nano /etc/apt/apt.conf.d/20auto-upgrades` and change the `1` to `0`. 


Notes:
 - By installing Ubuntu Server, the raspi will not have a GUI desktop. You can optionally install it, but we generally dont need it for our drones/rovers. 
 - Using Ubuntu Core means you have to use single sign on, so I think server is better. 


## STEP 2: Install Docker

Either you can follow instructions on the official Docker page, or use the convenience script. Here, we use the convenience script:
1. `curl -fsSL https://get.docker.com -o get-docker.sh`
2. `sudo sh get-docker.sh`

To test it, run `docker run hello-world`

## STEP 3: Extra steps

See https://github.com/dasc-lab/sd-images/blob/main/raspi4-ubuntu-20/raspi4-ubuntu-20-setup.md for additional steps that might be necessary. 

