# Prusawire Config for Klipper

## Installation

### Some Basic Assumptions

This guide assumes you are running an out-of-the-box installation of [MainsailOS](https://docs-os.mainsail.xyz/) on a Raspberry Pi.
We do not recommend using KIAUH, as this tends to be over-zealous with how it configures your machine.

This config requires Klipper from 2025-04-06 or later. The homing macros use the `SET_HOMED` parameter of `SET_KINEMATIC_POSITION`, which older builds silently ignore. Check the Klipper version on Mainsail's Machine page and update Klipper from there if it is older.

This config requires a toolhead board: LDO Nitehawk-SB V1 or V2, or BTT SB2209 USB or CAN. The extruder, hotend, fans and probe are all wired to it. A stock-wired MK3 with the extruder on the mainboard is not supported.

For installing MainsailOS (and with that, Klipper) for the first time, please refer to their [installation guide](https://docs-os.mainsail.xyz/getting-started/raspberry-pi-os-based).

### Upgrading the Einsy Rambo to Klipper - Read this!
Installing Klipper on the Einsy Rambo board is possible, with extra steps. Follow the guide published by the awesome folks at [MyRigs3D](https://myrigs3d.com/blogs/infos/revive-your-prusa-mk3s-with-klipper-1-5-flash-bootloader)! We recommend Method 2.

**Note:** Some users have reported problems using `avrdude` with the latest Raspberry Pi OS version (bookworm). If you experience error messages from avrdude complaining about gpio ports being busy, please try using the bullseye version of Raspberry Pi OS instead, available as "Raspberry Pi OS (Legacy)" in Raspberry Pi Imager.

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

- `PRINT_START` draws a [Squiggly Purge](https://github.com/mjonuschat/voron-mods/tree/main/Squiggly%20Purge) line by @mjonuschat before every print. It is included by the standard macros; the purge length is the `PURGE_LENGTH` parameter in `macros/print_start_end.cfg`.

- Set the rotation_distance within the printer.cfg based on the pulley size

- Run PID calibration on your hotend:
```shell
PID_CALIBRATE heater=extruder TARGET=250
```

- If running boards other than the Einsy, PID calibrate your heated bed:
```shell
PID_CALIBRATE heater=heater_bed TARGET=110
```

- Calibrate the probe Z offset. Home, then without moving the toolhead run:
```shell
PROBE_CALIBRATE
```
  Follow the paper test with `TESTZ`, then `ACCEPT` and `SAVE_CONFIG`. Homing leaves the toolhead at bed center, which is also where the mesh takes its zero reference, so do not move before calibrating.

## Slicer Setup

Start G-code (PrusaSlicer, OrcaSlicer):
```
PRINT_START BED=[first_layer_bed_temperature] EXTRUDER=[first_layer_temperature]
```

End G-code:
```
PRINT_END
```

Enable object labels so adaptive meshing and purging know where the print is. PrusaSlicer: Print Settings > Output options > Label objects: Firmware-specific. OrcaSlicer: Others > Exclude objects.

## Toolboard Wiring Notes

The two BTT SB2209 configs are derived from BTT's documentation and schematics and have not yet been run on a printer. If you build one, please report how it went on the Positron 3D Discord.

### BTT SB2209 USB

- Part cooling fan to FAN1, hotend fan to FAN2, as in BTT's sample config. Set each port's voltage jumper to match the fan.
- SuperPINDA signal to pin 3 of the PROBE header (gpio22). Its open-collector output only pulls the line low, so the direct MCU pin is safe.
- E3D PZ Probe signal to pin 5 of the PROBE header (gpio21), the buffered 5V tolerant input. The PZ Probe idles at its 5V supply, which the unbuffered pin 3 does not tolerate.
- The Omron goes on the IND port and a filament sensor on the ENDSTOP port, as before.

### BTT SB2209 CAN

- CAN needs a bridge on the host side, either a mainboard flashed in USB-to-CAN bridge mode or a U2C, and a `can0` interface. Follow https://www.klipper3d.org/CANBUS.html, then find the board's UUID with `~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0` and put it in the `[mcu sb2209can]` section.
- Part cooling fan to FAN1, hotend fan to FAN2, the same ports as on the USB board. BTT's CAN sample config has these two swapped; this config keeps one rule for both boards.
- SuperPINDA on the Proximity port. A filament sensor on the three-pin Endstop header.

### LDO Nitehawk-SB

If the board drops off USB after every FIRMWARE_RESTART and needs a power cycle, it has the V1.5 USB adapter board. Remove R6 and R7 as described in the [Prusawire FAQ](https://prusawire.positron3d.com/faq).

## Sensorless Homing

If you are running the Einsy board, congrats, you are now done.

For the BTT SKR Mini E3, some further tuning likely needs to happen. Refer to [this guide](https://gist.github.com/clee/9108f7717defce8b1222698f816def0a#finding-the-right-stallguard-threshold) by clee
on setting the correct stallguard threshold.

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

### TMC Auto Tune
[TMC Autotune](https://github.com/andrewmcgr/klipper_tmc_autotune) by @andrewmcgr

TMC Autotune is a Klipper extension for automaticly configuring and tuning TMC drivers. To fully use TMC Autotune, you will need to know the motor constants on each motor. For common motors, review the [motor_database.cfg](https://github.com/andrewmcgr/klipper_tmc_autotune/blob/main/motor_database.cfg) and search for the motor you have. If your motor does not show up on that list you will need to find the data sheet and create a custom motor as seen in [User-Defined Motors](https://github.com/andrewmcgr/klipper_tmc_autotune?tab=readme-ov-file#user-defined-motors).

### Klipper Shake&Tune plugin
[Klipper Shake&Tune plugin](https://github.com/Frix-x/klippain-shaketune/tree/main) by @Frix-x

Shake Tune allows you to visualize the harmonics of your machine and to quickly troubleshoot mechanical issues.

### External USB Mounting
[USB External Mount](https://github.com/DrumClock/mount_copy/tree/main) by @DrumClock

Install to allow external USB per @MattChu, who's progress can be tracked at: [Dont Click Me](https://projectshametracker.page/)
