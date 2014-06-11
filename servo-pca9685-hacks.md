# Getting the most out of your Servo Module

The servo module is one of the most versatile modules you can use with Tessel. The [chip at its center is actaully an adjustable PWM controller designed to control LEDs](http://www.nxp.com/documents/data_sheet/PCA9685.pdf) that we repurposed to be a servo driver. What this means is that it's easy to use for lots of applications besides just driving servos. Here are a few of the things it can do:

* Command and provide power to RC/hobby servos of all shapes and sizes
* Control control arbitrary actuators through the use of an external [motor controller](http://en.wikipedia.org/wiki/Motor_controller)
* Provide full control (brightness, blink speed, blink frequency) of small LEDs directly
* Provide full control of large numbers of LEDs with the help of a transistor
 

...And with a little work, it can do all of these things at once.

## Technical notes
**or "things you can exploit or may need to work around"**

* The power and ground on the Servo Module are shared
* The [barrel jack](http://www.cui.com/product/resource/pj-202a.pdf) is rated to 2.5A, but the [headers](http://media.digikey.com/PDF/Data%20Sheets/Sullins%20PDFs/z%20RzCzzzSzzN-RC,%20ST,11635-B.pdf) are rated to 3A apiece
* The duty cycle of each of the 16 channels is individually adjustable
* The PWM frequency is common for the entire chip (all channels)
* The drive strength of the chip is 25mA sink, 10mA source per channel at 3.3V
* The phase of each channel is adjustable via I2C commands, but not presently through the `servo-pca9685` library
* The library provides functions which allow you to control the max and min duty cycle of each individual channel. [This script](https://github.com/tessel/servo-pca9685/blob/master/examples/calibrate.js) is super handy for finding your specific actuator/controller's bounds.

### Commanding servos

If you completed the FRE you already know how to do this!

The only limiting factor for how many servos you can control through the Servo Module is the way in which power is delivered to the servos. As discussed above, the barrel jack on the Module is rated to 2.5 Amps, but power can be provided in a handful of ther ways:

* To the headers themselves, thereby bypassing the barrel jack
* Inline with the servo if the servo allows it/if you do some custom wiring
* At the servo if the hardware allows it, which is common for high-power servos such as [this one](http://www.robotshop.com/en/invenscience-torxis-i00600-12v-high-torque-servo-motor.html)

*Note:* Because the power rail for the Servo Module is shared by all 16 channels, make sure that all of your servos can run off the same voltage. Alternativey, isolate the power supply pins for each of your servos if you want to power them from different sources.

If you want two servos to baheve identically, it's ok to share the signal wire.

### Control of arbitrary actuators

By default and on its own, the Servo Module is configured to control any*thing* that has a 3-pin "servo" connector. If the actuator you want to control doesn't have such a connector, it can probably still be controlled using the Servo Module in conjunction with an external [motor controller](http://en.wikipedia.org/wiki/Motor_controller).

Controllers come in all shapes, sizes, actuator/motor types, power ratings, etc. and use a variety of different input control signals, and any of them can be used with Tessel with varying amounts of external components. The three most common "bins" into which control signals fall are

* Servo-style PWM inputs. These are the easiest to interface with the Servo Module because you literally just plug them right in and go
* Analog voltage inputs. Interfacing with these will require some combination of external circuitry and/or that the Servo Module's PWM frequency be set sufficiently high.
* Digital communication of some kind. UART is common, but typically would require a [level shifter circuit like this one](https://www.sparkfun.com/products/12009) in order to interface with the controller's logic levels.

#### Servo-style inputs

These are the easiest to use because, with a little [calibration](https://github.com/tessel/servo-pca9685/blob/master/examples/calibrate.js), they just *work*. Plug in the motor controller's input cable to the Tessel (be sure that the ground and signal wires are connected), provide the controller with power, and go.

#### Analog input

These controllers can be controlled either through the Servo Module or using Tessel's built-in PWM pins. What follows is true for either option.

It's not uncommon to find motor controllers which map an analog voltage to motor speed. Typically these mappings take the form of "move at a velocity proportional to (V<sub>reference</sub> / 2) - V<sub>command</sub>", where V<sub>reference</sub> is a fixed voltage, often 5V. In that case, a command voltage of 2.5V would make the motor stop and 0V and 5V would correspond to full speed in opposite directions.

In order to have the Servo Module produce an "analog" voltage, you'll likely need to put a [low-pass filter (LPF)](http://en.wikipedia.org/wiki/Low-pass_filter) on the Servo Module's output pin and will want to change the PCA9685's PWM frequency from its default of 50Hz to something on the order of 100kHz (or higher). [The simplest LPF is a resistor in between the Servo Module signal output and the motor controller's signal input and a capacitor between the motor controller's signal input pin and ground](http://en.wikipedia.org/wiki/Low-pass_filter#Electronic_low-pass_filters).

*Note:* If you would like to be able to control both servo-style motors/motor controllers and analog-input controllers, the PWM frequency must remain unchanged at 50Hz. In order to allow for a low PWM frequency, the resistor and capacitor values must be large (on the order of 100k Ohms and 10uF). This will provide a smooth speed response (read: steady but slow to change from one speed to another). If servo-style control is not also required, it is recommended that the PWM frequency be set very high and that the RC values be small (you may even be able to get away without them entirely).
