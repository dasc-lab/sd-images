# sd-images
images and instructions for loading Ubuntu onto a sd card, for use in Jetson Nanos and Raspberry Pis

See the appropriate files in the folders above for docs and example files


# Issues
1. The cloned image does not allow ssh from latop nor does it itself does ssh into the laptop. It freezes indefinitely. However, ssh from one rpi to another is not an issue and even roslaunch from laptop to start a node on rpi works but it freezes on normal ssh operation. This issue seems to be related to debian and a solution is posted here https://github.com/guysoft/OctoPi/issues/294. Simply add following to /etc/ssh/sshd_config file
```
IPQoS 0x00
```
