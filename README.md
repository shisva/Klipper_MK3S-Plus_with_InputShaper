WORK IN PROGRESS
-
Known Issues: PRINT_START Macro is currently configured to use KAMP, which isn't part of these instructions yet. Without KAMP, purge line won't be created. Pending whether we'll use other code or recommend KAMP.

As of 3/15/2024 - Current discussion and live help can be found here: https://discord.com/channels/431557959978450984/1205590905378312293

# Klipperized Input Shaping for the MK3S/+, a simplified guide for a powerful upgrade
Tired of seeing everyone upgrading to MK4s and feeling like you're missing out on the action? This project is an attempt to create simple, easy-to-follow documentation for performing an inexpensive, simple, and powerful upgrade to your Prusa MK3S/+. Originally based on work by dz0ny (https://github.com/dz0ny/klipper-prusa-mk3s).

WARNING: This upgrade involves the use and installation of Klipper, which is a significantly different experience than Prusa's firmware. Think Klipper = Custom Android, vs. Prusa = Apple iOS. Klipper is powerful, but can be potentially dangerous if misconfigured. We take no responsibility for damage or injury.

**There are several reasons you should try this upgrade:**
1. **Speed** - As you may know, Input Shaping can allow for substantial speed and print quality improvements to your printer. It's all the rage right now in the 3D Printing community.
2. **Low Cost** - It's inexpensive, requiring just $50-100 in hardware, which is a reasonable savings compared to the Prusa MKS3.5 Upgrade Kit.
3. **Simplicity** - We have structured our instructions and documentation to make this as simple as possible, despite the fact that Klipper is involved, and historically has been thought of as difficult to implement. Klipper is very powerful, and can be simple when implemented correctly.
4. **Easy to Return to Stock** - Undoing this upgrade and returning to stock Prusa firmware takes just a few steps, and can be completed in under 15 minutes.
5. **No Waste** - Your MK3S/+ Printer will not collect dust. Many in the Prusa community recommend either buying an MK4 while selling or keeping your MK3S. This upgrade allows you to keep your printer, while substantially increasing speed, and potentially increasing print quality.

**How will we do this?**

We will implement Input Shaping on your MK3S/+ with the use of Klipper. "YIKES, Klipper?! Isn't that super complicated?" No, while Klipper can be difficult to use, we've put in the work for you to allow for a simple  installation and configuration of Klipper. That doesn't necessarily mean it'll be easy, as implementing Klipper's Input Shaping can reveal hardware misconfigurations.

**What if I need to return to stock?**

Undoing this upgrade and returning to stock Prusa firmware takes just a few steps, and can be completed in under 15 minutes. Please see the section "Reverting to Stock Prusa Firmware".

## Hardware Required

- Prusa MK3S or MK3S+
  - The configuration files for this project are intended for use with the STOCK hotend and extruder. Other hotends and extruders listed below are possible to use with a modified configuration, but are out of the scope of this repository. For example configurations, please see the dz0ny repository, upon which this project is based: https://github.com/dz0ny/klipper-prusa-mk3s
  - Possible hotends: Stock, Dragon ST, Dragon HF, Revo, many others are supported by Klipper itself.
  - Possible extruders: Stock, Bondtech BMG, many others are supported by Klipper itself.
  - As with Prusa firmware, non-stock extruders require more extensive modification of the configuration.
- Raspberry Pi Zero 2 W - [https://amzn.to/3ODvStE](https://amzn.to/3TzhrIL)
  - Raspberry Pi 3B+/4/5+ will also work.
  - Raspberry Pi Zero W is not recommended.

- USB Type B male to USB Type A male cable (Came with your Prusa)
- USB Type A female to MicroUSB male converter (if using a Pi Zero 2 W, included in kit linked above)
- KUSBA: Klipper USB Accelerometer (enables easier Input Shaping calibration) - [https://amzn.to/4bzgAzA](https://amzn.to/49QFR79)
  - Optional, but recommended.
  - Connects to the Rpi via USB
  - NOTE 3/1/2024 - 3D-printed KUSBA mounts online worked but required modification to fit, the nozzle-mounted accelerometer could work better.
- KUSBA Alternative: Nozzle-mountable USB Accelerometer - Untested by us: [https://amzn.to/4cg55gQ ](https://amzn.to/3Vgk6t9)

## High-level Procedure
**Do not print using PrusaSlicer until Step 7!**

1) Install MainsailOS to your Raspberry Pi
2) Create configuration using the Primary Configuration Files in this repo
3) Flash firmware
    - Use USB method, serial method may only be necessary for some users
4) Perform Config Checks here, ESPECIALLY CALIBRATE PID: https://www.klipper3d.org/Config_checks.html (end-stops not applicable)
5) Customize PrusaSlicer for Klipper
    - Add MK3.5 printer to PrusaSlicer configuration
    - Add a custom printer for Klipper to configuration (Bed images and vector files are found in `C:\Program Files\Prusa3D\PrusaSlicer\resources\profiles\PrusaResearch`)
    - For Print Setting Presets, and Filament Setting Presets - Using the MK3.5 system presets, go to Dependencies > "Detach from System Preset"
    - Rename the newly detached presets, you can now use this for your Klipper Printer Profile
    - Add start and end code to your new custom printer profile in Printer Settings, under "Custom G-Code":
  Start Code
  ```yml
  M190 S0 ; Prevents prusaslicer from prepending m190 to the gcode interfering with the macro
  M109 S0 ; Prevents prusaslicer from prepending m109 to the gcode interfering with the macro
  PRINT_START EXTRUDER_TEMP=[first_layer_temperature] BED_TEMP=[first_layer_bed_temperature]
  ```

  End Code

  ```yml
  PRINT_END
  ```
6) Follow Ellis' Guide for primary tuning steps (NOT OPTIONAL): https://ellis3dp.com/Print-Tuning-Guide/articles/index_tuning.html
7) Perform Input Shaper calibrations and measurements - Revealed several hardware misconfigurations/issues for me personally
8) Re-do Pressure Advance Calibration post-IS

#
#
#
#

(Detailed instructions under review - FOLLOW THESE DETAILED STEPS AT YOUR OWN RISK)

## Step 0. Pre-Check and Expectations

- Watch this YouTube video for a basic introduction into Klipper: https://www.youtube.com/watch?v=iNHta6zljoM
- Depending on your familarity with Klipper, expect this full process (including tuning, which is the most time intensive) to take anywhere from 5-20 hours. This range is large because it depends on your own familarity with tuning procedures that Prusa normally takes care of for you. It can also be on the high side if hardware misconfiguration ends up contributuing to issues such as bad vibrations in your printer which will make Input Shaping calibration difficult until addresses. Please be prepared for this.
- Throughtout this process, it is recommended you reference 
- Get Z offset value from your current firmware (Menu -> Calibration -> Z-offset), you will need it for the Klipper config.
- Your bed needs to be perpendicular (based on Prusa XYZ Calibration results). An uneven printer means you need to fix the physical hardware first and re-visit the assembly instructions. If not you will have to do the skew calibration before printing or you risk crashing your nozzle to the bed.
- Some of the instructions will be direct links to external guides, which would not make sense to add here, particularly Ellis' 3D Printing guide.

## Step 1. Install MainSailOS on your Raspberry Pi

- We will use MainSailOS as it provides you with all of the components you'll need on your Raspberry Pi. There are other methods, but this is the most simple.
- Please follow this external guide: https://docs-os.mainsail.xyz/

## Step 2. Connect to Raspberry Pi via PC and Configure Klipper

1. Using a web browser, navigate to the URL you selected for your printer, such as `mainsailos.local`.
2. Once loaded, this will be your primary interface for connecting to your Raspberry Pi and Printer in one place. Remember, the Pi will be running the printer just like the Einsy board did on your Prusa.
3. Navigate to Machine. In the Root dropdown, select "config". This is the location of Klipper's critical configuration files.
4. In the Update Manager on the right, update everything. DO NOT UPDATE THESE IN THE FUTURE UNLESS YOU ARE CERTAIN THEY WILL NOT BREAK YOUR CONFIGURATION.
5. In the Config folder, use the files found in this repository's Primary Configuration Files to replace the existing files: https://github.com/charminULTRA/Klipper-Input-Shaping-MK3S-Upgrade/tree/main/1%20-%20Primary%20Configuration%20Files
   - NOTE: Either re-name Printer.Template.cfg, or copy/paste the contents into your own printer.cfg.
6. Once all the files have been updated, hit the Power icon at the top right, and select "Restart".

## Step 3. Connect Prusa MK3S/+ physical printer, and flash the Klipper firmware to your printer

1. Plug in the USB Type B male to USB Type A male cable, between your Raspberry Pi and your Prusa printer.
2. We will now be running some commands that will flash the Using SSH (please Google for appropriate methods depending on your PC's operating system), connect to your Raspberry Pi and log in.
3. Run the following commands one by one:
```yml
cd ~/klipper/
make menuconfig
```
Within the configuration menu, the firmware should be compiled for the AVR atmega2560. To use via USB, in "make menuconfig" select "Enable extra low-level configuration options" and select **UART0** when you plan to connect via the USB. Continue by running the following command in order to determine your printer's unique port ID:
```yml
ls /dev/serial/by-id/*
```
After this, run the following one by one. Be sure to update the YOUR_UNQIUE_ID with the printer's unique USB port ID.
```yml
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/YOUR_UNIQUE_ID
sudo service klipper start
```

If this process fails, it is possible that you may need to connect via the Serial port, which is a more involved process, requiring some hardware modification. TO BE UPDATED FOR INSTRUCTIONS OR LINKS

## Step 4. Perform configuration checks

1. See this page for Klipper's configuration checks: https://www.klipper3d.org/Config_checks.html
   - End stops are not applicable
2. PID Tuning is REQUIRED, just like with your Prusa factory firmware. Once you hit Save Config, copy paste the values at the bottom of your printer.cfg into the relevant upper sections.
    
## Step 5. Customize PrusaSlicer for Klipper
1. Add MK3.5 printer to PrusaSlicer configuration from the Configuration Wizard
2. Add custom printer in the Configuration Wizard, use whatever name you like.
   - Firmware Type = Klipper
   - Bed Shape = 250mm x 210mm
   - Import Bed Texture and Model Files - Located in `C:\Program Files\Prusa3D\PrusaSlicer\resources\profiles\PrusaResearch`
   - Max Print Height = 210mm
4. Copy the factory Prusa printing profiles:
   - For "Print Settings" Presets, and Filament Setting Presets - Using the MK3.5 system presets, go to Dependencies > "Detach from System Preset"
   - Rename the newly detached presets, you can now use this for your Klipper Printer Profile.
   - Add start and end code to your new custom printer profile in your custom printer's "Printer Settings", under "Custom G-Code". You will not use the factory start and end g-code.
  Start Code
  ```yml
  M190 S0 ; Prevents prusaslicer from prepending m190 to the gcode interfering with the macro
  M109 S0 ; Prevents prusaslicer from prepending m109 to the gcode interfering with the macro
  PRINT_START EXTRUDER_TEMP=[first_layer_temperature] BED_TEMP=[first_layer_bed_temperature]
  ```

  End Code

  ```yml
  PRINT_END
  ```
## Step 6. Tuning
1. Follow Ellis' Guide for primary tuning steps (NOT OPTIONAL): https://ellis3dp.com/Print-Tuning-Guide/articles/index_tuning.html
2. This step is a critcal part of implementing Klipper and cannot be skipped
3. For Pressure Advance and Extrusion Multiplier - It is recommended to add these values in to each individual Filament Profile in PrusaSlicer. PA can be added using the Custom G-Code menu, such as `SET_PRESSURE_ADVANCE ADVANCE=.055`
4. Once you finish tuning, try printing some test prints using the STOCK MK3S+ PROFILES.

## Step 7. INPUT SHAPING :)
1. To set some expectations, it is important that you run your input shaping tests on a very sturdy surface. Use of foam or any cushioning under your printer is not recommended, as it can skew the results, and you won't be using it after imlementing IS anyway. This process, through the resonance measurements, can also reveal potential issues with hardware misconfiguration, such as loose components, loose belts, etc, which can be time consuming to troubleshoot, but will pay off later.
2. KUSBA instructions: https://github.com/xbst/KUSBA/blob/main/Docs/v2-Rampon-Firmware.md
   - Installing the Linux MCU is NOT necessary...we use the MCU on the KUSBA
4. Links for KUSBA mounts
   - Nozzle mount (requires screws) https://github.com/xbst/KUSBA/blob/main/Mounts/M6_KUSBA_Mount.stl
   - Fan mount https://www.printables.com/model/495459-kusba-accelerometer-universal-mount
   - Other mounts: https://www.printables.com/search/models?q=kusba&ctx=models
5. It is recommended to follow this video for Input Shaping: https://www.youtube.com/watch?v=OoWQUcFimX8
6. You can also refer to  more technical Klipper guidance here: https://www.klipper3d.org/Measuring_Resonances.html

## Step 8. Re-do Pressure Advance after finishing your IS setup
8) Self Explainatory

## THE END! Happy printing.




