# Voron CAN bus guide (BETA)

## This is a BETA guide. - File an issue for anything that is missing or incorrect.

## Overview

CAN (Controller Area Network) bus is a standard that allows microcontrollers and devices to talk to each other without a host computer.  Using CAN bus on your Voron, can reduce wiring significantly and eliminate cable chains for an umbilical configuration.

Note: In this document, I will describe how to set up CAN bus for Voron Trident with StealthBurner and the Bondtech LGX Lite extruder.

![CANbus](https://user-images.githubusercontent.com/1135694/162856506-6330400c-a5a5-4562-9a6e-6aa23f4b3afb.jpg)

## Required hardware
- Huvud 0.61 CAN bus toolhead PCB
- 120 Ohm SMD resistor
- CANable Pro (preferred) or Waveshare RS-485/CAN HAT for Raspberry Pi
- Raspberry Pi 3 or greater (cannot use Pi 0W or Pi 0W2)
- CAN bus cable
- Mechanical probe (Klicky, Euclid, Unklicky, etc...)
- StealthBurner
- Bondtech LGX Lite (other extruders will work but will require different mounts)
- Various mounts

# Mounts

https://github.com/GiulianoM/LGX_Lite_Stealthburner_CW2_style_mount (Use the ECRF version even if you are NOT using ECRF.)

# X and Y Endstop Relocation

- https://github.com/hartk1213/MISC/tree/main/Voron%20Mods/Voron%202/2.4/Voron2.4_Y_Endstop_Relocation
- https://github.com/hartk1213/MISC/tree/main/Voron%20Mods/Voron%20Trident/Trident_Y_Endstop_Relocation

# Huvud CAN bus toolhead PCB, 120 Ohm resistor, flashing Klipper, StealthBurner LEDs.

### Toolhead PCB

https://github.com/bondus/KlipperToolboard

## PINs
|Name|Pin|
|---|---|
|Step|PB3|
|Dir|PB4|
|Enable|PB5|
|UART TX|PA9|
|UART RX|PA10|
|Thermistor 1|PA0|
|Thermistor 2|PA1|
|Heater|PA6|
|Fan 0|PA8|
|Fan 1|PA7|
|Endstop 1|PB10|
|Endstop 2|PB11|
|Endstop 3|PB12|
|ADXL CS|PB1|
|Diag for TMC2209?|PA15|
|SWDIO|PA13|
|CLK|PA14|

### 120 Ohm resistor

![image](https://user-images.githubusercontent.com/1135694/162861007-ee1593db-96d3-4d6c-a6a3-03a693e07703.png)


### StealthBurner LEDs

![image](https://user-images.githubusercontent.com/1135694/162858974-d1b0f9a2-3e10-4860-9d04-e1ebaf53817c.png)

# CAN bus adapter for Pi


It is recommended to use a Canable or Canable Pro adapter.  

https://canable.io

![image](https://user-images.githubusercontent.com/1135694/183714658-b94c5667-212e-4580-b198-0a6295e95575.png)


The Waveshare RS485 CAN HAT will drop connections even with a RPi 4.

https://www.waveshare.com/rs485-can-hat.htm
Note:  Newer Waveshare boards have a 12M clock and not a 8M!

![image](https://user-images.githubusercontent.com/1135694/162860544-40ec0002-9426-4fb1-b7a4-35897b54e892.png)


# Cable

You will need an umbilical cable with four 20 AWG conductors.  Ideally, two of the conductors are twisted (for CAN-L and CAN-H) and the overall cable has a shield.  Igus makes one but I find it to be too large and stiff to use as an umbilical.  Instead I use a non-twisted cable from Molex.  Some users using Ethernet cable.  It is critical that you use 20 AWG wire for power and ground!

![image](https://user-images.githubusercontent.com/1135694/162861039-06db6c08-f04a-4e2b-9474-34562b75e560.png)

![image](https://user-images.githubusercontent.com/1135694/162861108-7658c4e3-04b1-4860-bede-5ebf159d42e1.png)

### Flashing Klipper to the PCB

To flash the Huvud PCB, you must connect a cable to the microUSB port on the Huvud.

You must put a jumper on the pins BOOT1 and 3.3V then power cycle the board to put it into bootloader mode (green LED will flash quickly.) Flash with the command "make flash FLASH_DEVICE=1209:beba"

![image](https://user-images.githubusercontent.com/1135694/164993081-ebe8b1d1-c330-4e2b-bc01-f979f5b142c8.png)

# CAN bus configuration

These values are for the CANable.  You may have to reduce the bitrate to 250000 and txqueuelen to 128 for the Waveshare adapter and even then you may have communication timeouts.

/etc/network/interfaces.d/can0
```
auto can0
iface can0 can static
    bitrate 500000
    up ifconfig $IFACE txqueuelen 1000
```

# Klipper configuration

printer.cfg
```
[mcu can_mcu]
canbus_uuid: xxxxxxxxxxxx ; where xxxxxxxxx is the unique ID of your specific CAN bus toolhead
```

To find the UUID of your CAN bus controller:

```~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0```

bondtech_lgx_lite.cfg
```[extruder]
step_pin: can_mcu:PB3
dir_pin: can_mcu:PB4
enable_pin: !can_mcu:PB5

## BondTech LGX Lite
rotation_distance:5.7
##	Update Gear Ratio depending on your Extruder Type
microsteps: 16
#200 for 1.8 degree, 400 for 0.9 degree
full_steps_per_rotation: 200
nozzle_diameter: 0.400
filament_diameter: 1.75
## In E0 OUT Position
heater_pin: can_mcu:PA6
sensor_type: PT1000 
sensor_pin: can_mcu:PA0 # TE0 Position
pullup_resistor:2200
min_temp: 10
max_temp: 270
max_power: 1.0
min_extrude_temp: 170
control = pid
pid_kp = 26.213
pid_ki = 1.304
pid_kd = 131.721
##	Try to keep pressure_advance below 1.0
pressure_advance: 0.0725
##	Default is 0.040, leave stock
pressure_advance_smooth_time: 0.040

##	In E4-MOT Position
##	Make sure to update below for your relevant driver (2208 or 2209)
[tmc2209 extruder]
uart_pin: can_mcu:PA10
tx_pin: can_mcu:PA9
interpolate: false
run_current: 0.5
#hold_current: 0.2
sense_resistor: 0.110
#stealthchop_threshold: 0
```


# Input Shaping (ADXL) configuration

```
[input_shaper]
shaper_type_x: zv
shaper_freq_x: 63.0

shaper_type_y: mzv
shaper_freq_y: 39.8

[adxl345]
#cs_pin: rpi:None
cs_pin: can_mcu:PB1

[resonance_tester]
accel_chip: adxl345
probe_points:
    175, 175, 20  # an example
```

    
# Troubleshooting

If you are getting communication timeouts when homing, try:
- Moving any USB peripherals to USB 3.x ports (on Pi 4 at least)
- Reduce baud rate from 500,000 to 250,000
- Increase txqueuelen from 128 to 1000 (if your adapter can handle it)
- Upgrade Pi to Pi 4B 
- Check that CANL and CANH conductors in your cable are a twisted pair.  The power/gnd does not need to be twisted.  Shield the entire cable.

# Upgrading

```cd klipper; python lib/canboot/flash_can.py -i can0 -f ~/klipper_canbus/out/klipper.bin -u xxxxxxxxxxxx```

where xxxxxxxxx is the unique ID of your specific CAN bus toolhead

# See Also

https://elinux.org/CAN_Bus

