# Prusawire Config for Klipper

## Installation

### Some Basic Assumptions

This guide assumes you are running an out-of-the-box installation of [MainsailOS](https://docs-os.mainsail.xyz/) on a Raspberry Pi.
We do not recommend using KIAUH, as this tends to be over-zealous with how it configures your machine.

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

## Input Shaper

Some defaults have been provided, but they are no doubt unsuitable for your exact machine. We recommend installing [ShakeTune](https://github.com/Frix-x/klippain-shaketune) for measuring resonances, and reading the [Klipper guide](https://www.klipper3d.org/Measuring_Resonances.html#max-smoothing) on understanding which value to choose.

In addition, 

### Y Axis Input Shaping

This requires an external accelerometer (eg LDO Input Shaper) to be mounted to your heated bed.

## Additional Useful Add-ins

### TMC Auto Tune
[TMC Autotune](https://github.com/andrewmcgr/klipper_tmc_autotune) by @andrewmcgr

TMC Autotune is a Klipper extension for automaticly configuring and tuning TMC drivers. To fully use TMC Autotune, you will need to know the motor constants on each motor. For common motors, review the [motor_database.cfg](https://github.com/andrewmcgr/klipper_tmc_autotune/blob/main/motor_database.cfg) and search for the motor you have. If your motor does not show up on that list you will need to find the data sheet and create a custom motor as seen in [User-Defined Motors](https://github.com/andrewmcgr/klipper_tmc_autotune?tab=readme-ov-file#user-defined-motors).

### KAMP
[KAMP](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging) by @kyleisah

KAMP or Klipper Adaptive Meshing and Purging is an add-in that allows for addaptive meshes and purges that are near the mesh. While mainline Klipper has already added the adaptive mesh aspect, the adaptive purge placement is useful.

### Klipper Shake&Tune plugin
[Klipper Shake&Tune plugin](https://github.com/Frix-x/klippain-shaketune/tree/main) by @Frix-x

Shake Tune allows you to visualize the harmonics of your machine and to quickly troubleshoot mechanical issues.

### Klipper Shake&Tune plugin
[USB External Mount](https://github.com/DrumClock/mount_copy/tree/main) by @DrumClock

Install to allow external USB per @MattChu, who's progress can be tracked at: [Dont Click Me](https://projectshametracker.page/)
