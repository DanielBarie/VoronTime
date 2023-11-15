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
- yes, we might break things by activating this separately from mainsail's os update (but i'd rather be safe than sorry)

## get the touch screen rotation right:
- see https://docs.ldomotors.com/en/guides/btt_43_rotate_guide
- we choose to edit the config via ssh: ```sudo nano /boot/config.txt```
- when disabling vc4-fkms-v3d we lose external hdmi? or maybe we lose external hdmi because we've rotated the display? (https://www.raspberrypi.com/documentation/accessories/display.html)

## fix security
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

## install Kipper Installation and Update Helper
- Mainsail comes with git installed.
- get kiauh: ```cd ~ && git clone https://github.com/dw-0/kiauh.git```
- run: ```./kiauh/kiauh```
- enter ```1``, you will probably be asked for sudo...
- enter ```5``` to install klipperScreen (takes a while because we need all the packages for a GUI)
- 


