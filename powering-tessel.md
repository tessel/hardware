## Overview

Tessel can be powered either via its Micro-USB port or through the plated through holes located next to the USB connector ("VIN headers"). The USB port on Tessel is designed to supply the device with 5 V (and only 5 V) from the PC host, a Micro-USB charging cable, a USB battery pack, or similar. The VIN headers are designed for all other power input sources.

This page covers the following:

*  An overview of the power plant's features and behavior
*  How to power Tessel through the USB port
*  How to power Tessel through the VIN headers
*  How to swap between external and USB power
*  Usage of the GPIO bank's VIN pin
*  Calculating your maximum output current at 3.3 V
*  Power consumption
*  Absolute maximum ratings

Before reading further, we strongly recommend that you familiarize yourself with the [Tessel hardware](https://github.com/tessel/hardware/blob/master/tessel-hardware-overview.md). 

## Power plant overview

![Circled: Tessel power plant](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/hardware_design_docs/pp.png)

The circuitry which powers the Tessel, collectively referred to as the power plant, has a number of features designed to make Tessel flexible, efficient, and robust. They include:

*  Reverse voltage protection to -15 V, which means that plugging power in backwards (positive supply to GND and GND to VIN) won't break the Tessel.
*  A maximum input voltage via the USB port of 5 V.
*  A maximum input voltage through the VIN headers of 15 V.
*  Automatic power source selection circuitry. The circuitry favors USB power when available and allows the Tessel to be "hot-swapped" between power sources without halting code execution or rebooting the device. Proper hot-swap procedures are outlined later in this document.
*  A resettable fuse to protect the VIN input path from short circuits (NOT applicable to power in via the USB port).
*  A DCDC switching regulator which provides the 3.3 V rail which powers the Tessel.
*  A maximum output current on the 3.3 V rail of 3 A. Note that this maximum is not possible for all input power sources (see the section on powering through the VIN headers), nor is it necessary for the vast majority of Tessellations.

## Powering over USB

![Circled: micro USB port](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/hardware_design_docs/usb.png)

The easiest way to power Tessel is through the USB port. At the risk of overgeneralizing, as long as your USB power source can supply at least 500 mA, your Tessellation will function properly. Any USB 2.0 port, 5 V micro USB charger, 5 V "cell phone charger battery", etc. will do. Applying more than 5 V to the Tessel via the USB port may damage or destroy the hardware. For permanent installations/fixtures, we recommend a wall-power adaptor rated to at least 1000 mA.

## Powering through the VIN headers

![Circled: VIN headers](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/hardware_design_docs/vin.png)

The VIN headers allow for maximum flexibility in powering the Tessel, allowing input voltages between ~3.1 V and 15 V and reverse protection to -15 V. We recommend a minimum of ~3.55 V to allow for ~0.35 V headroom for the DCDC converter and power selection circuitry. Lower input voltages will mean that the 3.3 V rail will sag (drop below the nominal 3.3 V). The Tessel will shut off if the 3.3 V rail drops below ~2.9 V.

If you intend to power Tessel off of a battery pack, we recommend soldering wires or headers to the VIN headers.

### PTC fuse

![Circled: PTC fuse](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/hardware_design_docs/ptc.png)

The VIN header power input path includes a resettable, positive temperature coefficient ("PTC") fuse which will trip at 3 A. The fuse is designed to protect the Tessel (and anything to which it is attached) from sinking excessive current during short circuit conditions. It was designed with applications involving LiPo batteries in mind, where short circuit currents could easily reach damaging levels.

When a Tessel experiences a short circuit condition, the PTC will heat up, and in doing so change its electrical properties in such a way that the Tessel will shut down. Once the short is removed, the Tessel will boot up normally. The whole cycle (short, PTC trip, short removal, bootup) can occur in less than ten seconds.

If your Tessel's fuse keeps tripping, please read the section on proper use of the VIN pin on the GPIO bank.

## Hot-swapping

The power plant includes circuitry which allows you to change power input sources on the fly. In order to minimize the likelihood that doing so power cycles the Tessel, follow these steps:

1.  With the Tessel on and running, plug in the power source to which you will be switching.
2.  Remove the original power source at the cleanest possible connection point first. In other words, for a USB-to-Micro-USB cable, unplug it on the PC side (full size USB). If you're using a wall-powered adaptor, unplug it from the wall if possible. For power coming in to the VIN headers, remove the positive side (VIN, not GND) first.
3.  Fully disconnect the original power source from the Tessel, if applicable.
4.  You're good to go!

## Usage of the GPIO bank's VIN pin

![Circled: GPIO bank VIN pin](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/hardware_design_docs/vin-gpio.png)

The VIN pin on the GPIO bank is not directly connected to the VIN header by the USB port. Instead, it is connected to the same voltage the source-switching circuitry supplies to the DCDC converter. Consequently, the voltage on this pin is typically just under 5 V when Tessel is powered via USB and just under the voltage applied to the VIN header in the absence of USB power (~3.4 V to 15 V). **This also means that the pin is only to be used as an output; powering a Tessel through this pin could destroy the Tessel.**

The voltage at the GPIO bank's VIN pin should only be used either as a reference (to an ADC or similar) or to power low-draw, non inductive loads. Do not use it to power motors, servos, solenoids, or similar devices. LEDs, buzzers, and most sensors are probably fine.

If the system includes an external source of power, that source should be used to power the system's other components directly, as opposed to routing all power through the Tessel. This arrangement mitigates all the risks explained above, as well as the likelihood that a failure of other devices in the system would hurt Tessel.

If you are unsure of how to proceed, check the forums (at this point a work in progress) for guidance or [email us](mailto:team@technical.io).

## Current available on the 3.3 V rail

The DCDC converter ([datasheet](http://www.ti.com/lit/ds/symlink/tps62132.pdf)) and related circuitry in the power plant conserve power and are rated to a maximum of 3 A. Therefore, the actual maximum output current depends on the voltage and current rating of the power supply and the efficiency of the DCDC converter (represented by the Greek letter eta):

![current math](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/hardware_design_docs/dcdc-iout.png)

E.g., a Tessel powered via USB (assuming 5 V and 500 mA max and ~90% efficiency of the DCDC converter) can provide a maximum of ~680 mA out on the 3.3 V rail.

As a general rule, 90% is a safe estimate for the converter's efficiency across all input voltages.

## Power consumption

Power consumption is tough to calculate because it depends so heavily on what modules you use and how. Here are some things to keep in mind:

*  P = I*V
*  Tessel consumes about 1 Watt when it has the CC3000 WiFi radio enabled and is running ```blinky.js```.
*  Modules are marked with their peak current draw, not average current draw.
*  Bug [Kevin](mailto:kevin@technical.io). He's working on optimizing our runtime to allow for low-power operation.

## Absolute maximum ratings

These are the numbers at which point things are really, truly guaranteed break.

*  Voltage to VIN header: 17.5 V
*  Voltage to USB 5 V input: 5.5 V
*  Reverse voltage protection: -20 V
*  Current on the 3.3 V rail: 4.2 A
*  Current through the VIN headers: 3.2 A
