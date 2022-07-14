# How to create a new Fridge device


1. [OS Installation](./#os-installation)
2. [Device Configuration](./#device-configuration)

## OS Installation
### Simple way

1. Flash the `pi-fridge.img` that is stored here(todo) to a MicroSD card.
2. Insert the MicroSD.

### Advanced way

Instructions based on [this kiosk guide](https://blog.r0b.io/post/minimal-rpi-kiosk/).

```sh
$ sudo apt-get update && sudo apt-get dist-upgrade -y

$ sudo apt-get install --no-install-recommends xserver-xorg-video-all xserver-xorg-input-all xserver-xorg-core xinit x11-xserver-utils chromium-browser unclutter

$ sudo raspi-config
# go to: Boot Options > Console Autologin
# also go to: System Options > Password, set a password here (and store it somewhere)

$ nano /home/pi/.bash_profile
```

```sh
if [ -z $DISPLAY ] && [ $(tty) = /dev/tty1 ]
then
  pause 10
  startx -- -nocursor
fi
```

```sh
$ nano /home/pi/.xinitrc
```

```sh
#!/usr/bin/env sh
xset -dpms
xset s off
xset s noblank

unclutter &
chromium-browser https://eestarz.nl/fridge/login \
  --window-size=800,480 \
  --window-position=0,0 \
  --start-fullscreen \
  --kiosk \
  --noerrdialogs \
  --disable-translate \
  --no-first-run \
  --fast \
  --fast-start \
  --disable-infobars \
  --disable-features=Translate \
  --overscroll-history-navigation=0 \
  --disable-pinch \
  --pull-to-refresh=2 \
  --disable-hang-monitor
```

`Alt+F4` to exit kiosk mode, `startx -- -nocursor` to start chromium manually

## Device Configuration

After you've installed the OS you can start connecting the device to the internet.

1. Boot the Pi.
2. If the pi does not have an internet connection yet:
    1. Let the pi boot
    2. Connect a keyboard
    3. Press `ALT+F4`. The terminal should appear
    4. Configure internet access:
        - Wi-fi (make sure a dongle is connected if Pi <4):
          - Simple SSID + password:
            1. `sudo raspi-config`, Network Options, Wi-fi
            2. Follow the steps
            3. Once done, press the right arrow to select the Back button, and then Finish.
          - Eduroam (PEAP):
            1. `sudo ifconfig wlan0 down`
            2. `sudo killall wpa_supplicant`
            3. `sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`
            4. Make sure the following lines are in the file:
              ```conf
              update_config=1
              country=NL
              
              network={
                ssid="eduroam"
                eap=PEAP
                key_mgmt=WPA-EAP
                phase2="auth=MSCHAPV2"
                anonymous_identity="anonymous@<domain>.nl" # check your school's instructions
                identity="<eduroam login email>"
                password="<eduroam login password>"
              }
              ```
              `CTRL + X, Y, Enter` to save.
            5. `sudo ifconfig wlan0 up`
            6. You can run `sudo wpa_supplicant -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf` to test the connection. It should say "Authentication successful"
        - Ethernet (only required if your network requires registering mac-addresses):
          1. `sudo ifconfig eth0`
          2. Note down the mac-address
          3. Register the mac-address with your network's portal
    5. `sudo reboot`
