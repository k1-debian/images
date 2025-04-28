# Debian on Creality K1 Control Board

⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️

**This will completely erase everything from your printer. It's strongly recommended to back up your `printer.cfg` and any other configuration files you've customized, as well as any files you don't want to lose.**

**This process has many quirks. You should read the entire guide first and decide if you're comfortable proceeding.**

**Most of this is untested. You're about to upload and customize software to a machine that can burn your house down. You should *really* know what you're doing.**

⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️

## Flashing the Image

You'll first need to flash the Debian-based image.

1. Download the latest one from [Releases](https://github.com/k1-debian/images/releases). You want the `.ingenic` file.
2. Get and install the Cloner tool: [https://github.com/Ingenic-community/Cloner/releases/tag/v2.5.18](https://github.com/Ingenic-community/Cloner/releases/tag/v2.5.18). **You need version 2.5.18 exactly—no older or newer.**
3. Connect your board to your computer via a Micro-USB cable (on Windows, install the drivers provided with the Cloner tool).
4. Load the `.ingenic` image into the Cloner tool and press Start. You should see four columns in the main table.
5. Reset the board into `usb boot` mode. With the Micro-USB port facing down, the buttons are on the top left. The top button is `mode`, and the bottom is `reset`. Press and hold `mode`, press and release `reset`, then release `mode`. You should see activity in the Cloner tool.
6. Wait. It takes a significant amount of time to upload the image.
7. Once the upload is complete, **disconnect the Micro-USB cable**.
8. If the upload was successful, you should eventually see the blue LED on the board blink in a `blink-blink-pause` pattern. If so, continue with configuring your Wi-Fi.

## Configuring Wi-Fi

There's no user interface available at this point (unless you use a USB-to-serial converter for the serial console), so Wi-Fi must be configured via a USB flash drive.

1. Get a FAT32-formatted USB flash drive. Any should work.
2. Place a file named `wlan0.conf` in the root directory of the flash drive. Its content should be (replace `<your network name>` and `<your wi-fi password>` with your actual details):

```
update_config=1
ctrl_interface=DIR=/run/wpa_supplicant GROUP=netdev

network={
 ssid="<your network name(SSID)>"
 psk="<your wi-fi password>"
}
```

3. Insert the flash drive into the front USB port and wait.
4. Check your router for the IP address assigned to the printer and make a note of it.

Now that the printer is on the network and you know its IP address, you can continue by connecting to it via SSH.

## Connecting via SSH

With the IP address of the printer, you'll need:

* **Username:** `printer`  
* **Password:** `123456`

Use your preferred SSH client to connect. The `printer` user has `sudo` privileges.

**Run the following command immediately after successfully connecting to the printer:**

```bash
sudo resize2fs /dev/mmcblk0p4
```

This expands the root partition, allowing you to install additional software.

## Installing KIAUH

To install KIAUH, clone the repository:

```bash
cd ~ && git clone https://github.com/dw-0/kiauh.git
```

## Installing Klipper

You have two options for installing Klipper, depending on your probe setup.  
If you're using the original `prtouch` (bed pressure sensor), choose Option 1 for a setup that closely resembles the original experience.  
If you've installed an aftermarket or different probe, Option 2 gives you a clean, unmodified Klipper install that's more compatible with third-party extensions, but you'll lose `prtouch` support.

If you're unsure, choose **Option 1**.

---

### Option 1: Creality Klipper

1. Download the correct KIAUH config:
```bash
curl -o ~/kiauh/kiauh.cfg  https://raw.githubusercontent.com/k1-debian/images/refs/heads/master/kiauh/creality-Klipper.cfg
```
2. Start KIAUH:
```bash
./kiauh/kiauh.sh
```
3. **Be sure to select v6!**
4. Install Klipper, Moonraker, Mainsail, KlipperScreen, and Crowsnest. **Do not install the Mainsail printer config.**
5. Install host MCU support. Follow the instructions here: [https://www.Klipper3d.org/RPi_microcontroller.html](https://www.Klipper3d.org/RPi_microcontroller.html)
6. Copy the default printer configs for your printer:
```bash
cp -f ~/Klipper/config/<YOUR PRINTER>/* ~/printer_data/config/
rm -f ~/printer_data/config/factory_printer.cfg
```

Check [this directory](https://github.com/k1-debian/K1_Series_Klipper/tree/master/config) to see what options are available and choose the correct one for your printer. Replace `<YOUR PRINTER>` with the appropriate folder name.

7. Reboot:
```bash
sudo reboot
```

8. Configure Crowsnest—use one of the many tutorials available online.

9. Calibration and test print:

To calibrate the printer, run:
```
PRINT_CALIBRATION
```
Then save the configuration:
```
SAVE_CONFIG
```

After this, you're ready to perform a test print.

### CFS

Provided firmware was extracted from CFS update and therefore contains all support in Klipper for CFS.

Although Klipper itself supports CFS, there is absolutely no support for CFS in any GUI tool (Mainsail / KlipperScreen) so you will need to use raw macros for operating CFS. You can use KlipperScreen option to create custom menus.


### Upgrade from non-CFS version

If you had previous version installed, you will first need to completely uninstall Klipper using KIAUH, than delete `~/Klipper` and `~/klippy-env` directories and than install Klipper again. You will not loose your `printer.cfg` this way. After completing the upgrade, you can delete `~/printer_data/config/creality` directory - it is no longer needed.

### Input shaping calibration

All dependencies for input shaping calibration is already included. Do not use KIAUH option to install input shaping dependencies, it will fail.

---

### Option 2: Vanilla Klipper

1. Download the correct KIAUH config:
```bash
curl -o ~/kiauh/kiauh.cfg  https://raw.githubusercontent.com/k1-debian/images/refs/heads/master/kiauh/vanilla-Klipper.cfg
```
2. Start KIAUH:
```bash
./kiauh/kiauh.sh
```
3. **Be sure to select v6!**
4. Install Klipper, Moonraker, Mainsail, KlipperScreen, and Crowsnest.
5. Create your own `printer.cfg`, calibrate your printer, and perform a test print.  
   You can use the original Creality configs as a reference: [https://github.com/k1-debian/K1_Series_Klipper/tree/master/config](https://github.com/k1-debian/K1_Series_Klipper/tree/master/config)  
   **Note:** These configs will not work directly with vanilla Klipper.

If you want to use the original bed sensor, this project mey be interesting for you: [https://github.com/cryoz/K1_tenso_manual/blob/main/README_ENG.md](https://github.com/cryoz/K1_tenso_manual/blob/main/README_ENG.md)