# Twofing Changes for SunFounder 10.1" Touchscreen

Twofing is originally a package by [Plippo](https://github.com/Plippo) that monitors for a two-finger touch on the desktop. Once that is detected, a right-click is emulated. [Sjlongland](https://github.com/sjlongland) did a ton of work to get this debian package work in place.

This fork is written to add compatibility with the RasPad and its ILITEK touchscreen. By installing this, several things will happen:

- new udev rules will be added for this display
- new xorg InputClass created for the RasPad display with some default values for calibration. (see calibration below)
- the twofing program will be placed into /usr/bin
- twofing is set up to be managed by systemd

There are a few other small changes, including a fix for reading input from device ID.

## Building

Use the package manager to install Twofing. Either use the .deb release, or compile from the source here.

To do that, you need to follow a few steps first. Install a few prerequisites so that twofing can both be built and function:



```bash
sudo apt-get update

sudo apt-get install \
build-essential libx11-dev libxi-dev x11proto-randr-dev \
libxrandr-dev libxtst-dev xserver-xorg-input-evdev \
libinput-tools dh-systemd debhelper xinput-calibrator

```
Once the dependencies and other things are installed, you can build the package with dpkg-buildpackage. Enter the source directory and use:

```bash
dpkg-buildpackage -us -uc -b -tc
```
This will churn for a bit. a lot of output will be displayed as the program is being built. If all ends well, you should see something like this:

```
dpkg-buildpackage: info: binary-only upload (no source included)
```
Check your user's home directory for the completed .deb package.

## Installing

Installation is pretty straightforward and with little fanfare.

```bash
dpkg -i twofing_0.7.4_armhf.deb
```
Once again, there will be some output here. Systemd should announce that it is linking the service for startup.

```bash
pi@raspberrypi:/home/pi $ sudo dpkg -i /home/pi/twofing_0.7.2_armhf.deb
Selecting previously unselected package twofing.
(Reading database ... 161748 files and directories currently installed.)
Preparing to unpack .../pi/twofing_0.7.2_armhf.deb ...
Unpacking twofing (0.7.2) ...
Setting up twofing (0.7.2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/twofing.service → /etc/systemd/system/twofing.service.
```

Now systemctl should return the status of twofing.service:

```bash
pi@raspberrypi:/etc/udev/rules.d $ systemctl status twofing.service
● twofing.service - Twofing Rightclick service
   Loaded: loaded (/etc/systemd/system/twofing.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2019-11-12 23:40:59 PST; 2min 8s ago
 Main PID: 1973 (twofing)
    Tasks: 1 (limit: 4915)
   CGroup: /system.slice/twofing.service
           └─1973 /usr/bin/twofing
```
This service should be able to start automatically at boot, giving your RasPad a right-click mouse functionality.

## Troubleshooting

*Problem*: I get an error when compiling
* Check that all prerequisite packages are installed
* Check that the correct permissions are being using (especially if you see an error similar to 'dpkg: error: requested operation requires superuser privilege'

*Problem*: The touchscreen calibration is horrifically off! What happened?
* Calibration. the file used for xorg is calibrated to my screen. naturally, through both wear and the manufacturing process, your screen's accuracy and dimensions will be off. If you find this to be too distracting, new calibration data can be obtained by using *x-input-calibrator*. The usage of that particular command is fairly simple:

1. Start the Raspad
2. From the interface, open a terminal window.
3. run `x-input-calibrator`. Your screen will begin a calibration process.
4. After the calibration process, the terminal window will show a chunk of config for Xorg. The **Option axis values** should be replaced in /usr/share/X11/xorg.conf.d/70-twofing.conf. Leave the rest alone if you can. If for some reason the defaults are blasted away, refer to this:

```bash
Section "InputClass"
        Identifier      "calibration"
        Driver "evdev"
        MatchProduct    "ILITEK ILITEK-TP Mouse"
        MatchDevicePath "/dev/input/event*"
        Option  "MinX"  "-188"
        Option  "MaxX"  "66235"
        Option  "MinY"  "-539"
        Option  "MaxY"  "65378"
        Option  "SwapXY"        "0" # unless it was already set to 1
        Option  "InvertX"       "0"  # unless it was already set
        Option  "InvertY"       "0"  # unless it was already set
EndSection


```
5. Reboot your Raspad.

## Contact
Feel free to contact me. I am active on Twitter ([@akh13](https://www.twitter.com/akh13)).

## Contributing
Pull requests are welcome. This is a very rough work-in-progress.

## License
[MIT](https://choosealicense.com/licenses/mit/)
