# Using the Servo Module with LEDs, big motors, and more

![the humble servo module](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/servo-module-tutorial/servomodule.jpg)

The servo module is one of the most versatile modules you can use with Tessel. The [chip at its center is actaully an adjustable PWM controller designed to control LEDs](http://www.nxp.com/documents/data_sheet/PCA9685.pdf) that we repurposed to be a servo driver. What this means is that it's easy to use for lots of applications besides just driving servos. Here are a few of the highlights:

* The procedure explained in the [FRE](start.tessel.io/modules/servo) can be used to command and provide power to RC/hobby servos of all shapes and sizes.
* Each channel of the module can be [calibrated](#calibrate-your-servomotor-controller) to more effectively use a servo's full range of motion, the full range of speeds/torques available from a specific motor controller, or the full range of brightnesses for an LED/a strip of many LEDs.
* Control of arbitrary actuators through the use of an external motor controller.
* Provide full control (brightness, blink speed, blink frequency) of small LEDs directly, or of large numbers/strips of LEDs with the help of a transistor


Below are a series of tutorials to help get you up and running. No matter what you want to do in the end, it's recommended that you take the time at the beginning to [calibrate your hardware as described below](#calibrate-your-servomotor-controller) to make sure it gets used to its full potential.

---

### Technical notes

The following is a list of useful information about the servo module. If you know your way around hardware, most of the tutorials can be deduced from these facts and numbers.

* The power and ground on the Servo Module are shared across all servos. Ground is common with the Tessel.
* The [barrel jack](http://www.cui.com/product/resource/pj-202a.pdf) is rated to 2.5A, but the [headers](http://media.digikey.com/PDF/Data%20Sheets/Sullins%20PDFs/z%20RzCzzzSzzN-RC,%20ST,11635-B.pdf) are rated to 3A apiece
* The duty cycle of each of the 16 channels is individually adjustable
* The PWM frequency is common for the entire chip (all channels)
* The drive strength of the chip is 25mA sink, 10mA source per channel at 3.3V
* The phase of each channel is adjustable via I2C commands, but not presently through the `servo-pca9685` library
* The library provides functions which allow you to control the max and min duty cycle of each individual channel. [This script](https://github.com/tessel/servo-pca9685/blob/master/examples/calibrate.js) is super handy for finding your specific actuator/controller's bounds.

## Calibrate your servo/motor controller

The protocol used to command servos is very lenient. The advantage to this is that it makes interfacing with such devices very easy. The cost is, or at least can be, precision. Each servo and motor controller will behave a little differently, and if you really want to command a precise, known position or speed, you should calibrate your device. Before reading further, you should familiarize yourself with the [servo protocol](http://www.rcheliwiki.com/Servo_protocol). This tutorial doesn't touch the PWM frequency because the servo module uses the servo protocol's standard 50Hz by default. Adjusting the PWM frequecy is covered in the [LED tutorial](LINK GOES HERE).

Open and run [calibrate.js](https://github.com/tessel/servo-pca9685/blob/master/examples/calibrate.js)/copy and save the code below using `tessel run ./your/path/to/calibrate.js`. The code allows you to find the PWM duty cycle numbers that correspond to full forward, full reverse, and the neutral point by typing numbers into the command line and watching the result.

```js
var tessel = require('tessel');
var servolib = require('../'); // Or 'servo-pca9685' in your own code

var servo = servolib.use(tessel.port['A']);

var servoNumber = 1; // Plug your servo or motor controller into port 1

// Reenable the console
process.stdin.resume();
console.log('Type in numbers between 0.0 and 1.0 to command the servo.');
console.log('Values between 0.05 and 0.15 are probably safe for most devices,');
console.log('but be careful and work your way out slowly.');
servo.on('ready', function () {
  // Start at a duty cycle of 10%
  var duty = 0.1;
  // Set the min and max duty cycle to 0% and 100%, 
  // respectively to give you the maximum flexibility  
  // and to eliminate math
  servo.configure(servoNumber, 0, 1, function () {
    // Move into the starting position
    servo.move(servoNumber, duty);
    // Enter command positions into the command line. Work 
    // your way outwards until the servo begins to stall 
    // (or the motor velocity remains constant). Note the
    // max an min values and use them in this *physical* 
    // motor/controller channel's `configure` call.
    // Once you configure the motor, arguments to `move` of
    // 0 and 1 will correspond to the minimum and maximum
    // PWM values given in the `configure` call, thereby
    // allowing you to easily command the actuator through
    // its full range of motion.
    process.stdin.on('data', function (duty) {
      duty = parseFloat(String(duty));
      console.log('Setting command position:', duty);
      servo.move(servoNumber, duty);
    });
  });
});
```

After you've plugged everything in and run the code, the servo might look something like this when commanded with the script's default 10% duty cycle:

![10% duty cycle](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/servo-module-tutorial/servo-0.1.jpg)

Play around with the console to try out different values. Work your way slowly outwards to minimize the risk of damaging the servo by stalling out, drawing too much current, and frying something. Your servo is stalling if you can hear it trying to move but failing to. This happens when the servo is under heavy load ("working really hard"), and will happen when the servo runs into its built-in mechanical stops at the limits of its range of motion. When it happens you should be able to hear the motor inside the servo complaining. 

For reference, I found that this particular servo had lower and upper limits of 0.0275 (2.75%) and 0.1225 (12.25%), which allowed it to move through its ful range of motion:

![servo at one extreme](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/servo-module-tutorial/servo-low.jpg)

![servo at the other extreme](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/servo-module-tutorial/servo-high.jpg)

Once you've found values that work nicely for a particular physical servo or motor controller, save them! The command [`servo.configure( whichServo, minPWM, maxPWM, callback() )`](https://github.com/tessel/servo-pca9685#api-servo-configure-whichServo-minPWM-maxPWM-callback-Sets-the-PWM-max-and-min-for-the-specified-servo) maps the given max and min PWM values to 1.0 and 0.0, respectively, thereby making it easy to command two different devices to equivalent positions.

In the case of the values I found, the call would look like this:

```
servo.calibrate(1, 0.0275, 0.1225, function () {//callback goes here});
```

I could then call `servo.move(1, 0)` to set the servo to the lower extreme and `servo.move(1, 1)` to move it to the other extreme.



