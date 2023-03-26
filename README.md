# dobson-star-tracker
This project aims to enable makers to motorize their dobson-style mounted telescopes using easily sourced hardware.

## Hardware

+ 2 Stepper Motors + drivers. I found that the size of my telescope (as pictured badly below) requires at least NEMA 17 with 1.2A current per phase. The altitude motor uses a (roughly) 5:1 gear ratio
+ Arduino with at least 6 Digital Pins. The instructions and example config work for an Arduino Mega with a RAMPS 1.4 shield
+ 3D printed motor mounts and gears
+ Optional: Two Buttons; Each requires one additional digital pin
+ Optional: A GPS module; Requires either a free TX/RX pin pair or two digital pins

## 3D Files

The printable files (stl and step format) can be found at: https://www.thingiverse.com/thing:3851307


## Installation

First of all, you will need the [Arduino IDE](https://www.arduino.cc/) and a few libraries:

* [TimerOne (on the Arduino Mega)](https://github.com/PaulStoffregen/TimerOne)
* [DueTimer (on the Arduino Due)](https://github.com/ivanseidel/DueTimer)
* [AccelStepper](https://www.airspayce.com/mikem/arduino/AccelStepper/)
* [FuGPS (should work without, if no GPS module is installed)](https://github.com/fu-hsi/FuGPS)
* [TimeLib](https://github.com/PaulStoffregen/Time)

Clone or download this repository and open the dobson-star-tracker.ino file in the Arduino IDE. The first thing you will need to set up are a few constants in the config.h file. Please read through the whole file and set everything according to your needs. When you initially build and upload the sketch without setting at least the `AZ_STEPS_PER_REV`and `ALT_STEPS_PER_REV` constants, the scope will not move since both of the values are set to 0. This is done to prevent the motors from moving unexpectedly and maybe damaging your telescope. Check the output of the Serial Monitor for more information. Initially, `DEBUG`, `DEBUG_SERIAL` and `DEBUG_STOP_ON_CONFIG_INSANITY` are enabled for useful output via the Serial Monitor. Once everything works correctly, you can disable them. For more information on how to connect the scope to Stellarium or how to use the display unit, check below.

## Connection to Stellarium

There are a few requisites for establishing a connection between the telescope and Stellarium.

### Arduino project
1. Set the `SERIAL_BAUDRATE` to 9600
2. Disable all of the DEBUG constants, so that the telescope does not send invalid commands to Stellarium. If Stellarium receives an invalid answer, it will wait for a few seconds before interacting with the telescope again. This either means, that it won't receive position updates from the telescope or that it won't send your commands. Setting a new position requires three commands sent by Stellarium. If one of them receives an invalid answer, Stellarium will not send the rest of the commands and your input will basically be ignored.
3. Close the Serial Monitor or Stellarium will not be able to connect to the telescope

### Settings in Stellarium
1. Open Stellarium and locate the "Telescope" tab. It should be in the lower control bar to the left of the time controls. If you can't find it, you may need to enable the "Telescope Control" plugin and restart Stellarium. Do so in the Configuration window in the last tab.
2. Click "Add" and set the following:
    * **Telescope controlled by:** "Stellarium, directly through a serial port"
    * **Name:** Choose one :)
    * **Connection delay:** Something around 0,1s, but should not matter too much
    * **Coordinate System:** J2000 (default)
    * **Start/Connect at startup:** Ticking the box means that Stellarium will try to connect to the telescope as soon as you open it. Use your preference.
    * **Serial port:** Select the same port as in your Arduino IDE
    * **Device Model:** Meade LX200 (compatible)
    * **Use field of view indicators:** Does not matter for controlling the telescope. Use your preference.
3. Confirm the settings with "OK"
4. Use "Start" to connect to the telescope
5. Wait a few seconds for it to show up on the screen and for it to settle down
6. The motors will not move until you align the telescope. To do so, follow this procedure:
   1. Choose your alignment star and select it in Stellarium
   2. Click "current object" in the telescope section
   3. Manually point the telescope at the selected star
   3. Confirm alignment by clicking "slew" and wait for the telescope on screen to point at the correct star
   4. The telescope is now aligned to that particular star and the stepper motors will be enabled
7. Now you can select a different star in Stellarium
8. Click "current object" in the telescope section
9. Click "slew" and watch the telescope move


## Wiring a RAMPS1.4

![Wiring without the display unit](docs/img/Wiring_No_Display.png)


## Wiring with the Display Unit

My example uses a version of the well-known [RepRap Discount Full Graphics Smart Controller](https://reprap.org/wiki/RepRapDiscount_Full_Graphic_Smart_Controller). The only connection between the two Arduino boards is via the two cables connected to RX2 and TX2 on the Display Unit. For more information + code for the display unit head over to [ThisIsJustARandomGuy/telescope-display-unit/](https://github.com/ThisIsJustARandomGuy/telescope-display-unit/).

![Wiring without the display unit](docs/img/Wiring_With_Display.png)


# Aligning using the display unit

If you have a display unit, you can use it to easily align the telescope. This works with/without Stellarium. To align the telescope when using a display unit, do the following:

1. Start the display unit and scope
2. Navigate to "Set Alignment" -> "Stars"
3. Look through the list and select a star that's visible in the sky
4. Once you click the star in the list, the motors of the telescope will turn off
5. Align the telescope to the star you have selected
6. Click again and confirm the alignment
7. The motors will turn on again and the telescope is now aligned.

## Serial Commands

+ :HLP# Print available Commands
+ Commands used by stellarium (you can use them as well)
  + :GR# Get Right Ascension
  + :GD# Get Declination
  + :Sr,HH:MM:SS# Set Right Ascension; Example: :Sr,12:34:56#
  + :Sd,[+/-]DD:MM:SS# Set Declination (DD is degrees) Example: :Sd,+12:34:56#
  + :MS# Start Move
  + :Q# Quit Move (Not Implemented)
+ Other commands
  + :TRK0# Disable tracking. This sets the telescopes isHomed member variable to false, so the motors stop moving
  + :TRK1# Enable tracking. The telescope will track whatever the target is
  + :STP0# Disable steppers permanently
  + :STP1# Enable steppers (after they were disabled using the STP0 command)
+ Debug commands
  + :DBGDSP# Send a status update to the display unit
  + :DBGDM[00-99]# Disable Motors for XX seconds
  + :DBGM[0-9]# Move to debug position X (see conversion.cpp; Later we will have a separate file with a star catalogue)
  + :DBGMIA# Increase Right Ascension by 1 degree
  + :DBGMDA# Decrease Right Ascension by 1 degree
  + :DBGMID# Increase Declination by 1 degree
  + :DBGMDD# Decrease Declination by 1 degree

## TODOs

The Most important TODOs are as follows (in no particular order)
+ Documentation
+ Equatorial Mounts: Adding equatorial mounts would be rather easy, but I don't own one so it's difficult to test. Maybe someone is interested in providing a PR?
+ Board compatibility: Out-of-the-box support for Arduino Mega and Arduino Due with their respective RAMPS shields (Mostly done, Mega + RAMPS 1.4 and Due + modified RAMPS 1.4 work)
+ Time keeping: Handle big swings which could happen due to GPS issues
+ EEPROM: Store time and location in EEPROM (Mega) or Flash (Due); Provide a simple API for doing so
+ Motor control: Turn On/Off permanently. Off for X seconds is already implemented as :DBGDM[00-99]#
+ SD card support: Loading a star atlas from a sd card (preferrably connected to the display unit, but direct connection should be possible too)