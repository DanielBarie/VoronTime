# VoronTime
It's Voron TIme.

Built with an LDO kit.

# Things to do after completing mechanics
Following https://docs.ldomotors.com/en/voron/voron2/wiring_guide_rev_c

## Get the printer into the local wlan
- Maybe do this from the imager (Ctrl-Shift-X), if not:
-  ```sudo raspi-config```
- Set appropriate Country Code
- edit ```/etc/wpa_supplicant/wpa_supplicant.conf```
- see https://wiki.ubuntuusers.de/WLAN/wpa_supplicant/

## Activate ssh
- either in the imager
- or via ```sudo raspi-config```or
- via placing ```ssh``` file in boot partition before inserting sd card into pi

## Activate unattended upgrades for OS
- ```sudo apt-get install unattended-upgrades```
- Fix stupid Raspberry Pi OS Update Naming: https://raspberrypi.stackexchange.com/questions/38931/how-do-i-set-my-raspberry-pi-to-automatically-update-upgrade
- yes, we might break things by activating this separately from mainsail's os update (but i'd rather be safe than sorry)

## Get the touch screen rotation right:
- see https://docs.ldomotors.com/en/guides/btt_43_rotate_guide
- we choose to edit the config via ssh: ```sudo nano /boot/config.txt```
- when disabling vc4-fkms-v3d we lose external hdmi? or maybe we lose external hdmi because we've rotated the display? (https://www.raspberrypi.com/documentation/accessories/display.html)

## Fix security
We need a password protected web interface
See https://forum.vorondesign.com/threads/protect-your-klipper-printer-mainsail-password-ssl-firewall-rules.469/
- ```apt-get install apache2-utils```
- ```sudo htpasswd -c /etc/nginx/0-passwords.txt <user name>```
- generate ssl/tls certificate: ```openssl req -x509 -nodes -days 36500 -newkey rsa:2048  -keyout /etc/nginx/0-snakeoil.key  -out /etc/nginx/0-snakeoil.crt```
- ```sudo nano /etc/nginx/sites-available/mainsail```
  - comment out ```listen 80 default_server```
  - insert authentication/ssl directives somewhere below [server]:
    ```
    listen 443 ssl default_server;
    ssl_certificate      /etc/nginx/0-snakeoil.crt;
    ssl_certificate_key  /etc/nginx/0-snakeoil.key;
    auth_basic "Velcome to Voron";
    auth_basic_user_file /etc/nginx/0-passwords.txt;
    ```
- ```sudo service nginx restart```

## Update Mainsail:
Just Update all (lower right corner)
![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/27ede033-c0b2-42bb-8d99-68b102502f1a)

## Install Klipper Installation and Update Helper and KlipperScreen
- Mainsail comes with git installed.
- get kiauh: ```cd ~ && git clone https://github.com/dw-0/kiauh.git```
- run: ```./kiauh/kiauh```
- enter ```1``, you will probably be asked for sudo...
- enter ```5``` to install klipperScreen (takes a while because we need all the packages for a GUI)
- enter ```b```, ```q``` to leave kiauh
- nice, Touchscreen Interface is online.

## Install Klipper Firmware on Octopus BTT Board
- https://docs.vorondesign.com/build/software/octopus_klipper.html
- make is already installed
- ```cd klipper```
- ```make clean```
- ```make menuconfig```
- we have:
  - STM32F446
- all other options as per above guide (only had to change clock to 12MHz)
- compile (```make```), gives firmware bin file
- as for booting the board in DFU mode: That's a huge pain. Sometimes it might work, sometimes it just doesn't for no obvious reason.
- So we go for putting the fimrware on the sd card.
  - don't forget to rename to ```firmware.bin```
  - use a sufficiently small SD card (32GB because of FAT vs exFAT)

## Install Configuration Files for Printer in Mainsail
https://docs.vorondesign.com/build/software/configuration.html
- get it from https://raw.githubusercontent.com/VoronDesign/Voron-2/Voron2.4/firmware/klipper_configurations/Octopus/Voron2_Octopus_Config.cfghttps://raw.githubusercontent.com/VoronDesign/Voron-2/Voron2.4/firmware/klipper_configurations/Octopus/Voron2_Octopus_Config.cfg
- rename to ```printer.cfg```
- upload mainsail -> machine -> config files -> upload:
![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/729b459f-72ff-4ccb-b0c7-51eb53b60356)
- our octopus board's id is ```usb-Klipper_stm32f446xx_52001F000551313133353932-if00``` (```ls /dev/serial/by-id```)
- edit printer.cfg:
  - let mainsail know about the Octopus board:
    ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/b873e335-564e-4fce-ac1c-863097cc9185)
  - let's have a 350mm printer:
    ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/1053158b-371a-410c-818f-e3f61b4c7b52)
   


- 
# Issues still open:
- check groundung of DIN rails
- check grounding of frame
- check 5V Octopus Adapter according to https://news.ldomotors.com/2023/01/19/octopus-power-adapter-issue/
- square gantry
- install lcd cover


