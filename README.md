# Debian on Creality K1 control board

⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️

**This will completely erase everything from your printer. It is a very good idea to backup your `printer.cfg` and other config files if you customized them and also any other files you don't want to lose**

**This process has many quirks. You should first read the whole guide and decide if you are comfortable going through it**

**Anything here is mostly untested. You are about to upload and customize software into the machine, that can burn your house down. You should really know what you are doing.**

⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️

## Flashing the image

You will first need to flash the Debian-based image. 

1. Get the latest one from [Releases]https://github.com/k1-debian/linux/releases). You want the `.ingenic` file.
2. Get and install the Cloner tool [https://github.com/Ingenic-community/Cloner/releases/tag/v2.5.18](https://github.com/Ingenic-community/Cloner/releases/tag/v2.5.18). **You need version 2.5.18 exactly, not older, not newer.**
3. Connect your board using the Micro-USB cable to your computer (and on Windows, install the drivers from the Cloner tool)
4. Load the `.ingenic` image into the cloner tool and press start. You should see 4 columns in the main table.
5. Reset the board into `usb boot` mode. Looking at the board Micro-USB down, they are on the top left. The top button is `mode` and the bottom is `reset`. Press and hold `mode`, press and release `reset` and than release `mode`. You should see activity in the Cloner tool.
6. Wait. It will take significant time to upload the image.
7. After the upload is complete, **disconnect the Micro-USB cable**.
8. If the upload was successful, you should eventually see a blue LED on the board blink in`blink-blink-pause` pattern. If so, continue with configuring your Wi-Fi.

## Configuring Wi-Fi

There is no way to configure the board now. The system is running but we have no usable user interface (except the serial console, for which you need usb-uart converter). The Wi-Fi configuration will therefore be done using USB flash disk.

1. Get FAT32 formatted USB flash drive. Any should work
2. Put a file named `wlan0.conf` into the root of the flash drive. The content of the file should be (don't forget to replace the SSID and password with yours):

```
update_config=1
ctrl_interface=DIR=/run/wpa_supplicant GROUP=netdev

network={
 ssid="<your network name(SSID)>"
 psk="<your wi-fi password>"
}
```

3. Insert the flash drive into the front USB port and wait.
4. Check in your router what IP address was assigned to the printer and mark it down.

After this, you should have your printer in the network and you should know its current IP address, you may continue with connecting to the printer using SSH.

## Connecting using SSH

You already know the printer's address. You will need:

* Username: `printer`
* Password: `123456`

Use your SSH client of choice and connect to the printer using this information. The `printer` user is allowed to use `sudo`.

**Run this command on the printer immediately after you successfully connect to the printer:**
```bash
sudo resize2fs /dev/mmcblk0p4
```

This will make room on the root partition so you may further install software.

## Installing KIAUH

This guide uses KIAUH to install Klipper.

1. Clone the repository
```bash
cd ~ && git clone https://github.com/dw-0/kiauh.git
```
2. Edit the following file: `~/kiauh/kiauh/components/moonraker/moonraker_setup.py`
```bash
nano ~/kiauh/kiauh/components/moonraker/moonraker_setup.py
```
Find a line that looks like this (should be around line 193):
```python
        install_python_requirements(MOONRAKER_ENV_DIR, MOONRAKER_SPEEDUPS_REQ_FILE)
```
and comment it out (so it looks like this)
```python
        # install_python_requirements(MOONRAKER_ENV_DIR, MOONRAKER_SPEEDUPS_REQ_FILE)
```
Not doing this will result in a very long installation of Moonraker (due to the limited amount of RAM)

## Installing Klipper

If you are not using the original Creality bed sensor - `prtouch`, because you have modified your printer with a different sensor, you can disregard this section and instead install the standard version of Klipper following many tutorials available on the Internet.

This here is for those, who want to use the `prtouch` "probe" originally installed on the K1 series printers.

1. Start KIAUH:
```bash
./kiauh/kiauh.sh
```
2. **Select that you want to use v6!**
3. On the main screen, select `Settings` and then `Set Klipper source repository`
3. Enter this as repository URL: `https://github.com/k1-debian/K1_Series_Klipper.git` and keep the default `master` branch.
4. Go back to the main screen and install Klipper, keep all the settings default.
6. After installation of Klipper, do not continue to install other tools, instead quit KIAUH.
7. Stop Klipper service:
```bash
sudo systemctl stop klipper
```
8. You will need to replace Klipper python env, as this is now created with Python 3.11 and we need it with Python 3.8. Use the following commands:
```bash
rm -rf klippy-env
virtualenv -p /usr/bin/python3.8 klippy-env
~/klippy-env/bin/pip install -U pip setuptools
~/klippy-env/bin/pip install -r ~/klipper/scripts/klippy-requirements.txt
```
9. You will need to build and enable the host mcu service.
Go to Klipper directory:
```bash
cd Klipper
```
run `menuconfig`
```bash
make menuconfig
```
For `Micro-controller Architecture` select `Linux process`, and keep the rest default. Save and exit.
Run to build and install:
```bash
make && make flash
```
Install and enable the service:
```
sudo cp ./scripts/klipper-mcu.service /etc/systemd/system/
sudo systemctl enable klipper-mcu.service
sudo systemctl start klipper-mcu.service
```
10. Go back to the home directory
```
cd ~
```
11. Copy the default printer configs for your printer:
```bash
cp -f ~/klipper/config/<YOUR PRINTER>/* ~/printer_data/config/
rm -f ~/printer_data/config/factory_printer.cfg
mkdir ~/printer_data/config/creality
```

Check [https://github.com/k1-debian/K1_Series_Klipper/tree/master/config](https://github.com/k1-debian/K1_Series_Klipper/tree/master/config) and see what directories are available and choose the right one for your printer (replace `<YOUR PRINTER>` placeholder with chosen directory name)


12. Start Klipper service again
```bash
sudo systemctl start klipper
```

At this point, you should have working Klipper. You should go back to KIAUH and install other services like Moonraker, Mainsail / Fluidd, Crowsnest ...

After installation of the web interface of your choice, open in - Klipper should be running. Continue with calibration and the first test print. You can also install and configure Crowsnest to get built-in camera working.

If you choose to install mainsail, do not add its config import to `printer.cfg`, it would interfere with Creality macros.

## Calibration and test print

To calibrate the printer, run the following gcode:
```
PRINT_CALIBRATION
```
and save by using
```
SAVE_CONFIG
```

After this, you should go and make a test print.