# Getting the most out of your Servo Module

The servo module is one of the most versatile modules you can use with Tessel. The [chip at its center is actaully an adjustable PWM controller designed to control LEDs](http://www.nxp.com/documents/data_sheet/PCA9685.pdf) that we repurposed to be a servo driver. What this means is that it's easy to use for lots of applications besides just driving servos. Here are a few of the things it can do:

* Command and provide power to RC/hobby servos of all shapes and sizes
* Control external motor controllers of all shapes and sizes (which can then control arbitrary actuators)
* Set the brightness of small LEDs directly
* Control the brightness of large numbers of LEDs with the help of a transistor
 

...And with a little work, it can do all of these things at once.

## Technical notes
**or "things you can exploit or may need to work around"**

* The power and ground on the Servo Module are shared
* The [barrel jack](http://www.cui.com/product/resource/pj-202a.pdf) is rated to 2.5A, but the [headers](http://media.digikey.com/PDF/Data%20Sheets/Sullins%20PDFs/z%20RzCzzzSzzN-RC,%20ST,11635-B.pdf) are rated to 3A apiece
* The duty cycle of each of the 16 channels is individually adjustable
* The PWM frequency is common for the entire chip (all channels)
* The drive strength of the chip is 25mA sink, 10mA source per channel at 3.3V
* The phase of each channel is adjustable via I2C commands, but not presently through the `servo-pca9685` library

### Commanding servos

If you completed the FRE you already know how to do this!

The only limiting factor for how many servos you can control through the Servo Module is the way in which power is delivered to the servos. As discussed above, the barrel jack on the Module is rated to 2.5 Amps, but power can be provided in a handful of ther ways:

* To the headers themselves, thereby bypassing the barrel jack
* Inline with the servo if the servo allows it/if you do some custom wiring
* At the servo if the hardware allows it, which is common for high-power servos such as [this one](http://www.robotshop.com/en/invenscience-torxis-i00600-12v-high-torque-servo-motor.html)

*Note:* Because the power rail for the Servo Module is shared by all 16 channels, make sure that all of your servos can run off the same voltage. Alternativey, isolate the power supply pins for each of your servos if you want to power them from different sources.

If you want two servos to baheve identically, it's ok to share the signal wire.

