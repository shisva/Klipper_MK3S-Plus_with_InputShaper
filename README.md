WORK IN PROGRESS - NOT READY

# Input Shaping for the MK3S/+, an easy method for a powerful upgrade
Tired of seeing everyone upgrading to MK4s and feeling like you're missing out on the action? This project is an attempt to create simple, easy-to-follow documentation for performing an inexpensive, simple, and powerful upgrade to your Prusa MK3S/+. Originally based on work by dz0ny (https://github.com/dz0ny/klipper-prusa-mk3s).


**There are several reasons you should try this upgrade:**
1. **Speed** - As you may know, Input Shaping can allow for substantial speed and print quality improvements to your printer. It's all the rage right now in the 3D Printing community.
2. **Low Cost** - It's inexpensive, requiring just $50-100 in hardware, which is a reasonable savings compared to the Prusa MKS3.5 Upgrade Kit.
3. **Simplicity** - We have structured our instructions and documentation to make this as simple as possible, despite the fact that Klipper is involved, and historically has been thought of as difficult to implement. Klipper is very powerful, and can be simple when implemented correctly.
4. **Easy to Return to Stock** - Undoing this upgrade and returning to stock Prusa firmware takes just a few steps, and can be completed in under 15 minutes.
5. **No Waste** - Your MK3S/+ Printer will not collect dust. Many in the Prusa community recommend either buying an MK4 while selling or keeping your MK3S. This upgrade allows you to keep your printer, while substantially increasing speed, and potentially increasing print quality.

**How will we do this?**

We will implement Input Shaping on your MK3S/+ with the use of Klipper. "YIKES, Klipper?! Isn't that super complicated?" No, while Klipper can be difficult to use, we've put in the work for you to allow for a simple  installation and configuration of Klipper.

**What if I need to return to stock?**

Undoing this upgrade and returning to stock Prusa firmware takes just a few steps, and can be completed in under 15 minutes. Please see the section "Reverting to Stock Prusa Firmware".

## Hardware Required

- Prusa MK3S or MK3S+
  - Compatible hotends: Stock, Dragon ST, Dragon HF
  - Compatible extruders: Stock, Bondtech BMG 
- Raspberry Pi Zero 2 W - https://amzn.to/3ODvStE
  - Raspberry Pi 3B+/4/5+ will also work, but tend to be more expensive
  - Raspberry Pi Zero W is not recommended, and may not work

- USB Type B male to USB Type A male cable (Came with your Prusa)
- USB Type A female to MicroUSB male converter (if using a Pi Zero 2 W, included in kit linked above)
- KUSBA: Klipper USB Accelerometer (enables easier Input Shaping calibration) - https://amzn.to/4bzgAzA
  - Optional, but recommended.
- (HOW DOES THE KUSBA CONNECT TO THE PI? I actually don't fully know) FIXME
-

(INSTRUCTIONS SECTION)


Steps I took (high level):
1) Installed MainsailOS
2) Created config (see my notes below for corrections)
3) Flashed firmware
4) Did sanity check and PID tuning steps (see my notes, need to mention we don't have end stops)
5) Calibrate mesh bed leveling from Heightmap menu. (NOTE TO SELF - Why isn't it ran automatically before a print? Is that a slicer thing I need to add, or a config item?
6) Performed PA using this guide: https://ellis3dp.com/Print-Tuning-Guide/articles/pressure_linear_advance/introduction.html - I need to add specific steps on where to place output of the results. NOTE: PRUSA ALREADY HAS PRE-SET RECOMMENDED "K-VALUES" ON A PER-FILAMENT-TYPE BASIS. We should use those instead if using PrusaSlicer, but I need to figure out where they're located so we can bring them over and allow users to access them easily.
7) Skipped any extruder calibration since we already have all of this data captured in dz0ny's config files.
9) NEXT = Skew Correction, try both methods.


MISC NOTES:
-Make sure Pause and Resume still work: https://ellis3dp.com/Print-Tuning-Guide/articles/useful_macros/pause_resume_filament.html
-I want to add KAMP - Klipper Adaptive Meshing Purging - The only problematic issue here is we need to stay away from the magnets on the print bed as they'll cause errors in readings.
-KAMP requires exclude object (https://www.klipper3d.org/Exclude_Object.html#exclude-objects) which I need to add to the template.
-Implement SKEW_PROFILE LOAD=my_skew_profile into start Gcode.
-I copied the Extruder 1 page's settings from MK3.5 into my custom printer in PrusaSlicer.
-2/12 - Current lack of a purge step in the Gcode causes extrusion problems! That's why that's so important. Need to figure out how to enable this.

Extra notes from Discord:
-Extruder steps are already configured in the dz0ny files
-Squish - I already have down from knowing my correct z-offset at 0.20mm first layer height
-EM is usually configured in the filament settings in PrusaSlicer, not something I've ever touched except with special PETG projects.
-PA - Done, but again I'd debate whether this is necessary if we already have recommended values from Prusa.
-Retraction - Also something I've never touched on my Prusa and had no issues, so I'm using the stock slicer preset values.
-PrusaSlicer - I can use the MK3.5 profiles if you copy them over properly, not difficult thanksfully.




(KUSBA Specific info gathering)
Discuss:
1) What is the KUSBA
2) Why we use this Rampon firmware
3) How this whole thing works lol
4) Link to their instructions: https://github.com/xbst/KUSBA/blob/main/Docs/v2-Rampon-Firmware.md
5) Include links for KUSBA mounts
   
   1) Nozzle mount (requires screws) https://github.com/xbst/KUSBA/blob/main/Mounts/M6_KUSBA_Mount.stl
   2) Fan mount https://www.printables.com/model/495459-kusba-accelerometer-universal-mount
   3) Other mounts: https://www.printables.com/search/models?q=kusba&ctx=models

  











 

CURRENT PROGRESS MARKER - ALL BELOW IS FROM ORIGINAL REPO

## Pre-Check

- Get Z offset value from your current firmware (Menu -> Calibration -> Z-offset), you will need it for the Klipper config.
- Your bed needs to be perpendicular (based on XYZ Calibration). If not you will have to do the skew calibration before printing or you risk crashing your nozzle to the bed.
- Read https://github.com/dz0ny/klipper-prusa-mk3s/blob/main/printer.template.cfg
- Read https://www.klipper3d.org/Installation.html#building-and-flashing-the-micro-controller

## Install
1. Install https://docs.mainsail.xyz/setup/mainsail-os to SDCard and RPI Zero 2 W
2. Connect as described in https://help.prusa3d.com/en/article/raspberry-pi-zero-w-preparation-and-installation_2180
3. Update all components under Machine tab, otherwise config might not be able to load
4. Clone config ```git clone https://github.com/dz0ny/klipper-prusa-mk3s.git ~/printer_data/config/klipper-prusa-mk3s```

  > If you are adding this configuration after installing Klipper via [KIAUH](https://github.com/th33xitus/kiauh), the directory might be different - typically following `~/[printer_name]/printer_data/config`, where `[printer_name]` is the name you selected during the Kiauh installation

5. Add the following to the to `moonraker.conf` to enable automatic updates

```yml
[update_manager prusa]
type: git_repo
origin: https://github.com/dz0ny/klipper-prusa-mk3s.git
path: ~/printer_data/config/klipper-prusa-mk3s
primary_branch: main
is_system_service: False
managed_services: klipper
```

2. Copy https://github.com/dz0ny/klipper-prusa-mk3s/blob/main/printer.template.cfg to `printer.cfg` in your klipper config (THIS IS INCORRECT, THERE WAS NO PRINTER.CFG WHEN I INSTALLED, SO IT'S DUPLICATE, RENAME, and MOVE or whatever.)
3. Adjust config to your hardware
4. Flash Klipper to your printer https://www.klipper3d.org/Installation.html#building-and-flashing-the-micro-controller (MOVE THEIR COMMANDS INTO MINE, TOO MUCH EXTRA INFO IN THIS LINK. All that's needed is Change Folders, Configuration from this repo in it, check the serial ID # command, then flash)

You will still need a USB cable as you cannot flash via an internal serial port. You can also use any other computer to compile your firmware.

To use this config, the firmware should be compiled for the AVR atmega2560. To use via serial, in "make menuconfig" select "Enable extra low-level configuration options" and select **serial1** (the RasPi serial) or **serial0** when you plan to connect via the USB.

To flash:
`make flash FLASH_DEVICE=/dev/serial/by-id/usb-Prusa_Research__prusa3d.com__Original_Prusa_i3_MK3_CZPX0620X004XK70128-if00` (NOT CORRECT, YOU NEED TO USE YOUR SPECIFIC PORT ID)

7. Print


## Nice things
[Klipper mesh on print area only install guide](https://gist.github.com/ChipCE/95fdbd3c2f3a064397f9610f915f7d02)


## Screenshots
![image](https://user-images.githubusercontent.com/239513/141822711-2818978e-2b87-4110-9b93-e5f489c9cdc7.png)
![image](https://user-images.githubusercontent.com/239513/141831204-89ced257-e67f-4b1f-add7-a3806cdd2617.png)
![image](https://user-images.githubusercontent.com/239513/141831245-11476041-240d-424a-8ff8-ffd8a03c08be.png)
![image](https://user-images.githubusercontent.com/239513/141831272-31b88652-ab3f-4978-8a4c-c54a83817dd1.png)
