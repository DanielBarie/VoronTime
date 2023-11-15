# VoronTime
It's Voron TIme.

Built with an LDO kit.

# Things to do after completing mechanics

## get the printer into the local wlan
- Maybe do this from the imager (Ctrl-Shift-X), if not:
-  ```sudo raspi-config```
- Set appropriate Country Code
- edit ```/etc/wpa_supplicant/wpa_supplicant.conf```
- see https://wiki.ubuntuusers.de/WLAN/wpa_supplicant/

## activate ssh
- either in the imager
- or via ```sudo raspi-config```or
- via placing ```ssh``` file in boot partition before inserting sd card into pi

## activate unattended upgrades for OS
- ```sudo apt-get install unattended-upgrades```
- Fix stupid Raspberry Pi OS Update Naming: https://raspberrypi.stackexchange.com/questions/38931/how-do-i-set-my-raspberry-pi-to-automatically-update-upgrade
- 

# get the touch screen rotation right:
- see https://docs.ldomotors.com/en/guides/btt_43_rotate_guide
- we choose to edit the config via ssh: ```sudo nano /boot/config.txt```
- when disabling vc4-fkms-v3d we lose external hdmi? or maybe we lose external hdmi because we've rotated the display? (https://www.raspberrypi.com/documentation/accessories/display.html)
- 
