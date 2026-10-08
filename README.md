# Prusawire Config for Klipper

## Installation

### Some Basic Assumptions

This guide assumes you are running an out-of-the-box installation of [MainsailOS](https://docs-os.mainsail.xyz/) on a Raspberry Pi.
We do not recommend using KIAUH, as this tends to be over-zealous with how it configures your machine.

For installing MainsailOS (and with that, Klipper) for the first time, please refer to their [installation guide](https://docs-os.mainsail.xyz/getting-started/raspberry-pi-os-based).

### Upgrading the Einsy Rambo to Klipper - Read this!
Installing Klipper on the Einsy Rambo board is possible, with extra steps. Follow the guide published by the awesome folks at [MyRigs3D](https://myrigs3d.com/blogs/infos/revive-your-prusa-mk3s-with-klipper-1-5-flash-bootloader)! We recommend Method 2.

**Note:** Some users have reported problems using `avrdude` with the latest Raspberry Pi OS version (bookworm). If you experience error messages from avrdude complaining about gpio ports being busy, please try using the bullseye version of Raspberry Pi OS instead, available as "Raspberry Pi OS (Legacy)" in Raspberry Pi Imager.

If the legacy OS method above doesn't work - please see the Troubleshooting section at the bottom of this README for detailed instructions on a workaround. 

### Process

- Run the following command from your SSH terminal

```shell
cd ~/
git clone https://github.com/Positron3D/prusawire-klipper-config.git ~/printer_data/config/prusawire
```

- Add this section to your moonraker.conf file

```ini
[update_manager prusawire-config]
type: git_repo
primary_branch: main
path: ~/printer_data/config/prusawire
origin: https://github.com/Positron3D/prusawire-klipper-config.git
managed_services: klipper
```

- Refer to the `printer.cfg.example` file on setting up your printer.cfg for the first time

- Set the rotation_distance within the printer.cfg based on the pulley size

- Run PID calibration on your hotend:
```shell
PID_CALIBRATE heater=extruder TARGET=250
```

- If running boards other than the Einsy, PID calibrate your heated bed:
```shell
PID_CALIBRATE heater=heater_bed TARGET=110
```

## Sensorless Homing

If you are running the Einsy board, congrats, you are now done.

For the BTT SKR Mini E3, some further tuning likely needs to happen. Refer to [this guide](https://gist.github.com/clee/9108f7717defce8b1222698f816def0a#finding-the-right-stallguard-threshold) by clee
on setting the correct stallguard threshold.

Run current and StallGuard threshold are tuned as a pair. If you change `run_current` for your motors (see the MOTORS section of `printer.cfg.example`), re-run the threshold tuning.

## Klipper Screen

For users that are using a TFT or HDMI screen, you will need to install Klipper Screeen [Link](https://klipperscreen.readthedocs.io/en/latest/) 

To install Klipper Screen, follow [this guide](https://klipperscreen.readthedocs.io/en/latest/Installation/)

## Input Shaper

Some defaults have been provided, but they are no doubt unsuitable for your exact machine. We recommend installing [ShakeTune](https://github.com/Frix-x/klippain-shaketune) for measuring resonances, and reading the [Klipper guide](https://www.klipper3d.org/Measuring_Resonances.html#max-smoothing) on understanding which value to choose.

### Y Axis Input Shaping

This requires an external accelerometer (eg LDO Input Shaper) to be mounted to your heated bed.

## Using Pi As MCU

If you encounter a Klipper error for mcu 'rpi': Unable to connect, follow the [Flashing RPI guide](https://www.klipper3d.org/RPi_microcontroller.html)

## Additional Useful Add-ins

### Squiggly Purge
[Squiggly Purge](https://github.com/mjonuschat/voron-mods/tree/main/Squiggly%20Purge) by @mjonuschat

For making fun shaped purges

### TMC Auto Tune
[TMC Autotune](https://github.com/andrewmcgr/klipper_tmc_autotune) by @andrewmcgr

TMC Autotune is a Klipper extension for automaticly configuring and tuning TMC drivers. To fully use TMC Autotune, you will need to know the motor constants on each motor. For common motors, review the [motor_database.cfg](https://github.com/andrewmcgr/klipper_tmc_autotune/blob/main/motor_database.cfg) and search for the motor you have. If your motor does not show up on that list you will need to find the data sheet and create a custom motor as seen in [User-Defined Motors](https://github.com/andrewmcgr/klipper_tmc_autotune?tab=readme-ov-file#user-defined-motors).

### Klipper Shake&Tune plugin
[Klipper Shake&Tune plugin](https://github.com/Frix-x/klippain-shaketune/tree/main) by @Frix-x

Shake Tune allows you to visualize the harmonics of your machine and to quickly troubleshoot mechanical issues.

### External USB Mounting
[USB External Mount](https://github.com/DrumClock/mount_copy/tree/main) by @DrumClock

Install to allow external USB per @MattChu, who's progress can be tracked at: [Dont Click Me](https://projectshametracker.page/)

# Troubleshooting
## Einsy Klipper - AVRDude GPIO Busy
The following workaround has been verified on both a Rapberry Pi 3 Model B and a Raspberry Pi 4. It is an extension of the Method 2 by [MyRigs3D](https://myrigs3d.com/blogs/infos/revive-your-prusa-mk3s-with-klipper-1-5-flash-bootloader) with the latest version of `avrdude`.

Pre-Requisites: 
- Pi OS: Latest or MainsailOS
- `avrdude.conf` is not modified
- GPIO pins are connected as described in Method 2

Steps:
1. SSH into your pi and install libgpiod-dev: via `sudo apt-get install -y libgpiod-dev`
2. Install the latest version of avrdude from github and build it on your machine (don't worry if you have the prev avrdude version still installed)
```
sudo apt-get install build-essential git cmake flex bison pkg-config libelf-dev libusb-dev libhidapi-dev libftdi1-dev libreadline-dev libserialport-dev

git clone https://github.com/avrdudes/avrdude.git
cd avrdude
./build.sh

cmake -D CMAKE_BUILD_TYPE=RelWithDebInfo -D HAVE_LINUXGPIO=1 -D HAVE_LINUXSPI=1 -B build_linux

# install linuxgpio if it's missing
sudo apt-get install linuxgpio

cmake --build build_linux

sudo cmake --build build_linux --target install
```
3. Reboot your pi `sudo reboot`
4. Do not edit the avrdude conf file directly, instead create a config file `nano ~/pi_1.conf`
5. Paste this into the config file - Note the mappings are now different (mosi -> sdo,  miso -> sdi)
```
programmer
    id = "pi_1";
    desc = "Raspberry Pi GPIO ISP programmer";
    type = "linuxgpio";
    connection_type = linuxgpio;
    prog_modes = PM_ISP;
    reset = 12;
    sck = 24;
    sdo = 23;
    sdi = 18;
;
```
**Important** - Because you've added this custom config file, the next commands must include both references to both.

**Important** - The following commands include a tag called `<hostname>`. You **should not directly copy paste this value**. This is a placeholder for **your** pi's hostname that you configured when you flashed the OS.

7. Run the updated firmware backup command with your hostname
```
sudo avrdude \
-C /usr/local/etc/avrdude.conf \
-C +/home/<hostname>/pi_1.conf \
-p m32u2 \
-F \
-c pi_1 \
-U flash:r:firmware_backup.hex:i \
-U eeprom:r:eeprom.hex:i \
-U lfuse:r:lowfuse:h \
-U hfuse:r:highfuse:h \
-U efuse:r:exfuse:h \
-U lock:r:lockfuse:h
```

8. Run the updated fuse command with your hostname
```
sudo avrdude -C /usr/local/etc/avrdude.conf -C +/home/<hostname>/pi_1.conf -p m32u2 -F -c pi_1 -U hfuse:w:0xD1:m
```
9. Download the prusa flashware `wget https://raw.githubusercontent.com/PrusaOwners/mk3-32u2-firmware/master/hex_files/DFU-hoodserial-combined-PrusaMK3-32u2.hex`

10. Run the updated flash firmware command with your hostname
```
sudo avrdude \
  -C /usr/local/etc/avrdude.conf \
  -C +/home/<hostname>/pi_1.conf \
  -p m32u2 \
  -F \
  -c pi_1 \
  -U flash:w:DFU-hoodserial-combined-PrusaMK3-32u2.hex \
  -U lfuse:w:0xFF:m \
  -U hfuse:w:0xD9:m \
  -U efuse:w:0xF4:m
```
11. Shut down the pi `sudo shutdown -r now` and disconnect all GPIO cables. 
