# Marlin for Artillery Sidewinder X2
![](https://img.shields.io/badge/Artillery%20Version-v2.0.9.1-blue)
![](https://img.shields.io/github/v/release/MarlinFirmware/Marlin?label=%20Marlin%20Version)

The stock firmware on the Artillery Sidewinder X2 is [Marlin](https://github.com/MarlinFirmware/Marlin), however [Artillery's version](https://github.com/artillery3d/sidewinder-x2-firmware) is out of date and missing some useful features.

Marlin is an open source project in active development and so with a little configuration the latest version can be made to run on any supported printer. 

In this repository you will find configuration for use with newer versions of Marlin on the Sidewinder X2 which consists of 2 files:
 - [`configs/updated/Configuration.h`](configs/updated/Configuration.h)
 - [`configs/updated/Configuration_adv.h`](configs/updated/Configuration_adv.h)

To save you having to manually figure out how to use these files I have written scripts to download the latest supported Marlin, apply the configuration and build the firmware to flash onto your printer.

Feel free to [open an issue](https://github.com/markrossington/sidewinder-x2-marlin/issues/new) if you think there are more sane defaults or find any bugs.

# Disclaimer
I offer no warranty, support or guarantees. Any changes to your 3D printer firmware and the consequences of these changes are your responsibility. If you do not understand what you are doing then I suggest you do not continue.

# Lets just do this

 1. Clone or [download a zip](https://github.com/markrossington/sidewinder-x2-marlin/archive/refs/heads/main.zip) of this repository
 2. Ensure you have Python 3.x installed. [Download here](https://www.python.org/downloads/).
 3. Run: `01-build.py` and compiled firmware should be in `output/firmware.bin`
 4. Ensure `dfu-util` installed, turn printer on and plug a USB cable into it. Run `02-flash.py`.

# Comparison of Differences from Stock

| Feature/Difference           | Stock              | This                      | Reference Variable                               | Rationale/Notes                                                                                          |
| ---------------------------- | ------------------ | ------------------------- | ------------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| Marlin Version               | 2.0.9.1            | 2.1.2 Bugfix              | N/A                                              | Less bugs, newer features, see [Marlin Releases](https://github.com/MarlinFirmware/Marlin/releases) page |
| Heater Minimum Temperature   | 5 C                | -5 C                      | HEATER_0_MINTEMP etc                             | 5 C is a bad default because it is a reasonable ambient temperature in a garage                          |
| Restore level after home     | Off                | On                        | ENABLE_LEVELING_AFTER_G28                        | Save having to enter `M420` to restore the mesh after home in slicer settings                            |
| Possible to detach servo     | Off                | On                        | SERVO_DETACH_GCODE                               | Add ability to send `M282` for lower power idle state                                                    |
| Kickstarting fans            | Off                | 100ms                     | FAN_KICKSTART_TIME                               | Allows the fan to spin up reliably                                                                       |
| Auto Aligning Steppers       | Off                | On                        | Z_STEPPER_AUTO_ALIGN                             | There are 2 Z axis steppers TBD check if this is needed                                                  |
| Input Shaping                | Off                | On                        | INPUT_SHAPING_X, INPUT_SHAPING_Y                 | Allows reduction in ringing and ghosting artifacts on prints induced by vibrations of the printer        |
| Adaptive Step Smoothing      | Off                | On                        | ADAPTIVE_STEP_SMOOTHING                          | Increases resolution of stepping for better print quality                                                |
| Linear Advance               | Off                | On                        | LIN_ADVANCE                                      | Maintains consistent pressure in the nozzle for cleaner corners/ other features which change speed       |
| Increased Arc support        | Low Res No Circles | Higher Res, allow circles | MIN_ARC_SEGMENT_MM, ARC_P_CIRCLES                |                                                                                                          |
| Internal Move Buffer         | 16                 | 64                        | BLOCK_BUFFER_SIZE                                | Increase buffer size providing there is SRAM available is a good thing                                   |
| Serial ASCII Buffer          | 4 bytes            | 32 bytes                  | BUFSIZE                                          |                                                                                                          |
| Serial Receive Buffer        | 128 bytes          | 2048 bytes                | RX_BUFFER_SIZE                                   | This makes printing with something like octoprint much smoother and less likely to stutter               |
| Advanced OK Command          | Off                | On                        | ADVANCED_OK                                      | Allow Marlin to respond with additional info when returning OK                                           |
| M600 Filament change feature | Off                | On                        | NOZZLE_PARK_FEATURE, ADVANCED_PAUSE_FEATURE      |                                                                                                          |
| Auto report position         | Off                | On                        | AUTO_REPORT_POSITION, M114_DETAIL, M114_REALTIME | some clients may use this                                                                                |
| Lower Case G-Code            | Off                | On                        | GCODE_CASE_INSENSITIVE                           | Why not                                                                                                  |
| Host Action Commands         | Off                | On                        | HOST_ACTION_COMMANDS                             | Allow more advanced features of octoprint at negligible cost to performance                              |
| Cancel Objects               | Off                | On                        | CANCEL_OBJECTS                                   | Useful mid print to continue with other projects if one may have lifted or otherwise failed              |

# Folder Structure

* configs
  * `updated` - The configuration to use on a new Sidewinder X2
  * `stock_artillery` - For reference only, to see which options are enabled on the stock firmware
* scripts
  * `01-build.py`
  * `02-flash.py`

# FAQ

  1. If you're going to flash a new firmware, why not just use [Klipper](https://www.klipper3d.org)?
     * Klipper will mean learning a whole new software ecosystem. This is a valuable learning project and **one I suggest to those who seek it**. However if you just want your printer to operate in the same way, but with better features: Marlin is the way to go. 
     * Feature wise Klipper used to be king, however the Marlin team are catching up with features such as linear advance and input shaping which are now both available. 
     * You absolutely need an additional computer (usually a Rapsberry Pi) constantly plugged into your printer. Not a problem for most of us who use Octoprint/OctoPi but still worth noting. Using Klipper your printer will not operate without this. 
     * The existing TFT screen will no longer work, there are great projects such as [Klipperscreen](https://klipperscreen.readthedocs.io) which will work with a screen plugged into your Raspberry Pi but this is extra hardware still. 
  2. Is this the firmware for the TFT?
     * No, this is the firmware for the mainboard. If you wish to update the firmware for the TFT then look at [Arillery's Download Page](https://www.artillery3d.com/pages/downloads) or for something new try [BigTreeTech's firmware](https://github.com/bigtreetech/BIGTREETECH-TouchScreenFirmware), though if you have a new style TFT then you might not be able to, see [the following issue](https://github.com/bigtreetech/BIGTREETECH-TouchScreenFirmware/issues/2391).
  3. Why are the scripts in Python and not X, Y or Z?
      * I wanted this to work on as many platforms as possible. Python allows for this and is often installed on many peoples machines anyway. It's worth noting that building Marlin doesn't work on a Raspberry Pi at this point in time, this is a limitation of the toolchains and nothing I can control.
  4. Why are no UI or SD card options configured?
      * The circuit board connected to the TFT display manages the UI and SD card on this printer. Marlin runs on a different board, the mainboard. For TFT related firmware see question 2.
  5. Why don't you have this new amazing feature installed?
      * I might not know about it, [open an issue](https://github.com/markrossington/sidewinder-x2-marlin/issues/new).

# Sources/Links

 - The amazing Marlin project: https://github.com/MarlinFirmware/Marlin
 - Official Marlin config for Sidewinder X2: https://github.com/MarlinFirmware/Configurations/tree/import-2.1.x/config/examples/Artillery/Sidewinder%20X2
 - Stock Binaries, if you wish to go back: https://www.artillery3d.com/pages/downloads
 - Stock source code: https://github.com/artillery3d/sidewinder-x2-firmware
 - Freakydude's Build: https://github.com/freakydude/Marlin
