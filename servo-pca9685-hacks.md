## Using the Servo Module with LEDs, big motors, and more

The servo module is one of the most versatile modules you can use with Tessel. The [chip at its center is actaully an adjustable PWM controller designed to control LEDs](http://www.nxp.com/documents/data_sheet/PCA9685.pdf) that we repurposed to be a servo driver. What this means is that it's easy to use for lots of applications besides just driving servos. Here are a few of the things it can do:

* Command and provide power to RC/hobby servos of all shapes and sizes
* Control control arbitrary actuators through the use of an external [motor controller](http://en.wikipedia.org/wiki/Motor_controller)
* Provide full control (brightness, blink speed, blink frequency) of small LEDs directly
* Provide full control of large numbers of LEDs with the help of a transistor
* Assorted other hacks
 

...And with a little work, it can do all of these things at once.

### Technical notes

The following is a list of useful information about the servo module. If you know your way around hardware, most of the tutorials can be deduced from these facts and numbers.

* The power and ground on the Servo Module are shared
* The [barrel jack](http://www.cui.com/product/resource/pj-202a.pdf) is rated to 2.5A, but the [headers](http://media.digikey.com/PDF/Data%20Sheets/Sullins%20PDFs/z%20RzCzzzSzzN-RC,%20ST,11635-B.pdf) are rated to 3A apiece
* The duty cycle of each of the 16 channels is individually adjustable
* The PWM frequency is common for the entire chip (all channels)
* The drive strength of the chip is 25mA sink, 10mA source per channel at 3.3V
* The phase of each channel is adjustable via I2C commands, but not presently through the `servo-pca9685` library
* The library provides functions which allow you to control the max and min duty cycle of each individual channel. [This script](https://github.com/tessel/servo-pca9685/blob/master/examples/calibrate.js) is super handy for finding your specific actuator/controller's bounds.

