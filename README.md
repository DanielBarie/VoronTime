# VoronTime
It's Voron Time. 

Built with an LDO kit, rev C.  

Issues along the way:  
- See safety
- The kit differs in some ways from the "official" build. Piecing together the correct instructions isn't the easiest thing to do.
- Chaoticlab CNC mod: Had to re-design/fix some parts.

# (planned) Modifications 
- separate 5V PSU for Raspberry Pi  
- Safety upgrade for heater control  
- Voron Tap, done
- CNC parts for extruder
- CNC parts for XY/Z belt tensioning (done, full CNC kit)
- AngryCam
- Z-Belt Covers with cable guides (done, modified with cutouts for linear rails)

# Things to do after completing mechanics
Following https://docs.ldomotors.com/en/voron/voron2/wiring_guide_rev_c

# SAFETY!
- Disconnect Heater Cables for Hotend and Bed Heating. (Pre-loaded BTT firmware may have these active, so Hotend and/or Bed may get hot).
- Even though having updated the firmware als per [LDO's instructions](https://docs.ldomotors.com/en/voron/voron2/wiring_guide_rev_c)  I had (at least two) cases where the heaters (Hotend, Bed) came on after startup without having activated theses in Klipper. Klipper performed a shutdown because the Hotend was outside its specified temperature range, heating continued anyway?!?) The Hotend (E3D) maxed out at 380°C. This will have to be rectified (probably by adding separate relays into Hotend/Bed Heater circuits which are controlled by Klipper's shutdown state).

First step is setting heater pins to a defined state at Klipper firmware startup (HE0 is hotend, HE1 is bed via SSR):
![grafik](https://github.com/user-attachments/assets/ecb594f2-874d-45ea-88d3-da5381bdd1bd)
![grafik](https://github.com/user-attachments/assets/ca41198c-71c1-4979-b178-74ebc76b5c27) (from https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-V1.0/blob/master/Hardware/BIGTREETECH%20Octopus%20-%20SCH.pdf)


Also make sure to have the right config for your Octopus Board. Ours is a v1.1 


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

## install useful tools
- ```sudo apt-get install mc```


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
- DO MAKE SURE TO SET PIN VALUES FOR STARTUP (!PA2,!PA3,!PB10,!PB11)
- compile (```make```), gives firmware bin file
- as for booting the board in DFU mode: That's a huge pain. Sometimes it might work, sometimes it just doesn't for no obvious reason.
- So we go for putting the fimrware on the sd card.
  - don't forget to rename to ```firmware.bin```
  - use a sufficiently small SD card (32GB because of FAT vs exFAT)

## Install Configuration Files for Printer in Mainsail
https://docs.vorondesign.com/build/software/configuration.html
- get it from https://raw.githubusercontent.com/VoronDesign/Voron-2/Voron2.4/firmware/klipper_configurations/Octopus/Voron2_Octopus_Config.cfghttps://raw.githubusercontent.com/VoronDesign/Voron-2/Voron2.4/firmware/klipper_configurations/Octopus/Voron2_Octopus_Config.cfg
- rename to ```printer.cfg```
- OR (better): Get the LDO kit version (because we've been using the Rev. C kit): https://github.com/MotorDynamicsLab/LDOVoron2/blob/main/Firmware/octopus-printer-rev-c.cfg  
- upload mainsail -> machine -> config files -> upload:
![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/729b459f-72ff-4ccb-b0c7-51eb53b60356)
- our octopus board's id is ```usb-Klipper_stm32f446xx_52001F000551313133353932-if00``` (```ls /dev/serial/by-id```)
- edit printer.cfg to get basic mechanics going:
  - let mainsail know about the Octopus board:  
   ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/fb708b1d-515e-4c67-879d-89c86cd4f61c)

  - let's have a 350mm printer:    
    ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/1053158b-371a-410c-818f-e3f61b4c7b52)
  - another axis with 350mm:  
    ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/1bcbf7b0-75b0-4a15-938a-e521de61cc84)
  - Z endstop:  
    ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/1d32f9b5-dbfb-4b64-9baa-e5f7cb346a70)

  - z1 stepper enable:  
    ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/4c6a18b9-b30b-47aa-aca4-9db96a04b621)

  - Extruder Control pins:
    ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/8fd454f4-98dc-435f-b837-27850e00bb64)
    

  - Nozzle Heater pin:  
    ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/e0bdde31-6856-4d70-ba5c-21dd7dc5251d)

  - Nozzle heater thermistor:  (change to ATC Semitec)
   ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/989720f9-ba14-4c05-ba66-5d2c05ed66f5)


  - Bed Heater:  (change thermistor to ATC Semitec)
    ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/e55f88ba-5110-48eb-9509-64891e77090f)

  - Probe config:
    ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/37ebdaa5-6241-4665-a0c0-318c16f9c3b8)

  - Gantry corners / probe points config:
    ![grafik](https://github.com/DanielBarie/VoronTime/assets/73287620/f324ba0e-792f-4cab-a1ee-6d7eea6470cc)

  -   Next: Check Bed Heating (complains about not heating quickly enough); Extruder Heating works
    

# Chaoticlab CNC Upgrade:
## XY Joints
Make sure to mount the toothed pulley on the black stand-off.  
![grafik](https://github.com/user-attachments/assets/b8f1047c-0181-4095-bcb3-7c05264423b8)    
Regular D2F endstop pod doesn't fit (Y Endstop doesn't reliably trigger). CNC Z Joints are smaller (not as wide) as the printed ones. To fix this, the endstop pod needs to be extended in y-direction (approx. 2mm). Part: ![modified/extended endstop pod](endstop_pod_extended_2.0.stl)  

## Voron Tap V2
- Belts need to be clipped.  
- Won't activate the X endstop. We need to print https://www.printables.com/de/model/781495-chaoticlab-voron-tap-v2-bracket-for-xy-endstop-pcb/files (TAP_XY_XL.stl) and mount this to the extension bracket intended for the endstop.

### LDO Toolhead PCB ChaoticLab V2 Wiring mismatch  
The LDO toolhead pcb probe connector has a different wiring, doesn't match Chaoticlab's Tap wiring harness: https://www.reddit.com/r/VORONDesign/comments/1aq32nw/chaotic_lab_cnc_tap_not_working/   
Get some JST XH 3 pin connectors, unpinning the one on the wiring harness didn't work for me. Cut it off.   
Original wiring harness connector on the toolhead side:  
![toolhead_connector_orig](https://github.com/user-attachments/assets/1a565f27-5c9c-4b24-bef2-4b9db69e4ff8)  
New wiring harness connector on the toolhead side:  
![toolhead_connector_new](https://github.com/user-attachments/assets/d5fa67d1-628b-4222-b161-fd4c43dcf1f7)  

# Can't connect to MCU?
There's a bug....
https://www.teamfdm.com/forums/topic/1892-beware-latest-klipper-update-breaks-symlink-to-mcu-on-usb-connected-devices/

# Z Belt Covers
Since there is LED strips installed we needed some way of getting the wiring into the electronics compartment. LDO provides STLs for modified Z Belt Covers, see https://github.com/MotorDynamicsLab/LDOVoron2/blob/main/STLs/z_belt_cover_a_led.stl  
These will do the job but there's better ones: https://mods.vorondesign.com/details/LzEFU0RDHXUarF7y69x2Q  
Our Z linear rails are too close, so We modified these with cutouts for the linear rails:
- [Z Belt Cover A, STEP-File](cable_routing_z_belt_cover_a_mod_v2_cutout_rail.4%20v1.step)
- [Z Belt Cover B, STEP-File](cable_routing_z_belt_cover_b_mod_v2_cutout_rail.4%20v1.step)

  
# Issues still open:
- check groundung of DIN rails
- check grounding of frame
- check 5V Octopus Adapter according to https://news.ldomotors.com/2023/01/19/octopus-power-adapter-issue/
- square gantry (done, heat soaking will need to be done after having installed panels)
- install lcd cover


