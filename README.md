# Klipperized Input Shaping for the MK3S/+, a simplified guide for a powerful upgrade

Based on https://github.com/charminULTRA/Klipper-Input-Shaping-MK3S-Upgrade

## High-level Procedure Overview (see detailed guide below)
**Do not print using PrusaSlicer until Step 6!**

1) Install MainsailOS to your Raspberry Pi
2) Create configuration using the Primary Configuration Files in this repo
3) Flash firmware
    - Use USB method, serial method may only be necessary for some users
4) Perform Config Checks 
5) Customize PrusaSlicer for Klipper
6) Follow Ellis' Guide for primary tuning steps (NOT OPTIONAL)
7) Perform Input Shaper calibrations and measurements - FYI, This step revealed several hardware misconfigurations/issues for me personally as a secondhand printer owner.
8) Re-do Pressure Advance Calibration after enabling IS



# Detailed procedures

## Step 0. Pre-Check and Expectations

- Watch this YouTube video for a basic introduction to Klipper: https://www.youtube.com/watch?v=iNHta6zljoM
- Depending on your familarity with Klipper, expect this full process (including tuning, which is the most time intensive) to take anywhere from 5-20 hours. This range is large because it depends on: 1) your own familarity with tuning procedures that Prusa normally takes care of for you, 2) your familiarity with Klipper, 3) your familiarity with tools like Raspberry Pi, SSH, and command line interfaces. It can also be on the high side if hardware misconfiguration contributes to issues such as bad vibrations in your printer, which will make Input Shaping calibration difficult until addresses. Please be prepared for this.
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
6. Edit the printer.cfg as necessary, instructions are inside of the printer.template.cfg file.
7. DO NOT edit the other files unless you know what you are doing.
8. Once all the files have been updated, hit the Power icon at the top right, and select "Restart".

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

If this process fails, it is possible that you may need to connect via the Serial port, which is a more involved process, requiring some hardware modification. The Serial port approach is out-of-scope for this guide. Please search Google for the method or reach out on the Klipper Discord: https://discord.com/channels/431557959978450984/1205590905378312293

## Step 4. Perform configuration checks

1. See this page for Klipper's configuration checks: https://www.klipper3d.org/Config_checks.html
   - End stops are not applicable
2. PID Tuning is REQUIRED, just like with your Prusa factory firmware. Once you hit Save Config, copy paste the values at the bottom of your printer.cfg into the relevant upper sections.
    
## Step 5. Customize PrusaSlicer for Klipper (DO NOT PRINT FROM THE SLICER UNTIL COMPLETING STEP 6!!)
1. Add MK3.5 printer to PrusaSlicer configuration from the Configuration Wizard
2. Copy the MK3.5 printer:
   - Go to Dependencies > "Detach from System Preset"
   - Rename the newly detached printer, you can now use this for your Klipper Printer Profile.
   - Delete ALL existing data in ALL Custom G-Code boxes! Important!
   - REPLACE the start and end code to your new custom printer profile in your custom printer's "Printer Settings", under "Custom G-Code". You will not use the factory start and end g-code. The "PRINT_START" and "PRINT_END" commands activate the identically-named macros in your Macros.cfg file. Macros are like containers for their own set of Gcode which can be referenced more easily.
5. Copy the factory Prusa printing profiles:
   - For "Print Settings" Presets, and Filament Setting Presets - Using the MK3.5 system presets, go to Dependencies > "Detach from System Preset"
   - Rename the newly detached presets, you can now use this for your Klipper Printer Profile.
   - CRITICAL: For the Printer Profile, change "G-Code Flavor" to "Klipper".
   - Delete ALL existing data in ALL Custom G-Code boxes! Important!
   - IMPORTANT: In Print Settings > Advanced - DISABLE "Arc Fitting" - G2/G3 should not be used with Klipper and is not necessary.


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
## Step 6. Tuning (Do not use PrusaSlicer until completeing this guide, or at least until getting to the EM step in this guide)
1. Follow Ellis' Guide for primary tuning steps (NOT OPTIONAL): https://ellis3dp.com/Print-Tuning-Guide/articles/index_tuning.html
2. This step is a critcal part of implementing Klipper and cannot be skipped
3. For Pressure Advance and Extrusion Multiplier - It is recommended to add these values in to each individual Filament Profile in PrusaSlicer. PA can be added using the Custom G-Code menu, such as `SET_PRESSURE_ADVANCE ADVANCE=.055`
4. Once you finish tuning, try printing some test prints using the STOCK MK3S+ PROFILES.

## Step 7. INPUT SHAPING :)
1. To set some expectations, it is important that you run your input shaping tests on a very sturdy surface. Use of foam or any cushioning under your printer is not recommended, as it can skew the results, and you won't be using it after implementing IS anyway. This process, through the resonance measurements, can also reveal potential issues with hardware misconfiguration, such as loose components, loose belts, etc, which can be time consuming to troubleshoot, but will pay off later.
2. It is recommended to follow this video for Input Shaping: https://www.youtube.com/watch?v=OoWQUcFimX8
3. You can also refer to  more technical Klipper guidance here: https://www.klipper3d.org/Measuring_Resonances.html
4. KUSBA installation instructions, excluding Steps 2.1 and 2.2: https://github.com/xbst/KUSBA/blob/main/Docs/v2-Rampon-Firmware.md
   - Steps 2.1 and 2.2: adxlmcu.cfg has already been provided in this repo, no need to add a new one for the KUSBA. I'm not familiar with other ADXL's, so these may need a different config.
   - May not be applicable, but if you run across any instructions elsewhere saying to install the Linux MCU, this is not necessary...we use the MCU on the KUSBA.
5. Non-KUSBA - Please Google for relevant instructions.
6. Links for KUSBA mounts
   - Nozzle mount (requires screws) https://github.com/xbst/KUSBA/blob/main/Mounts/M6_KUSBA_Mount.stl
   - Fan mount https://www.printables.com/model/495459-kusba-accelerometer-universal-mount
   - Other mounts: https://www.printables.com/search/models?q=kusba&ctx=models


## Step 8. Re-do Pressure Advance after finishing your IS setup
8) Self Explainatory

## THE END! Happy printing.

## Reverting to Stock Prusa Firmware
1. Connect the USB cable from your printer to your PC
2. Download Prusa's firmware from their website: https://help.prusa3d.com/downloads
3. From PruasSlicer, Configuration > Flash Firmware. Select your file and flash.



