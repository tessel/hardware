# Using the Servo Module with LEDs, big motors, and more

**Read this whole section before moving on.**

The servo module is one of the most versatile modules you can use with Tessel. The [chip at its center is actually an adjustable PWM controller designed to control LEDs](http://www.nxp.com/documents/data_sheet/PCA9685.pdf) that we repurposed to be a servo driver. What this means is that it's easy to use for lots of applications besides just driving servos.

![all the motors](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/servo-module-tutorial/motor_materials.jpg)

Everything in the image above and in the list below can be done with the servo module.

* [Command and provide power to RC/hobby servos of all shapes and sizes](http://start.tessel.io/modules/servo)
* [Calibrate each channel to get the most out of the hardware](#calibrate-your-servo). This includes a servo's full range of motion, the full range of speeds/torques available from a specific motor controller, or the full range of brightnesses for an LED/a strip of many LEDs.
* [Control arbitrary actuators through an external motor controller](#Arbitrary-actuators).
* [Control LEDs of all shapes and sizes, including their brightness, blink speed, blink frequency](LINK COMING SOON).


Below are a series of tutorials to help get you up and running. **No matter what you want to do in the end, take the time at the beginning to [calibrate your hardware as described below](#calibrate-your-servo) to make sure it gets used to its full potential.**

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

## Calibrate your servo

The protocol used to command servos is very lenient. The advantage to this is that it makes interfacing with such devices very easy. The cost is, or at least can be, precision. Each servo and motor controller will behave a little differently, and if you really want to command a precise, known position or speed, you should calibrate your device. Before reading further, you should familiarize yourself with the [servo protocol](http://www.rcheliwiki.com/Servo_protocol). This tutorial doesn't touch the PWM frequency because the servo module uses the servo protocol's standard 50Hz by default. Adjusting the PWM frequecy is covered in the [LED tutorial](LINK GOES HERE). The process for calibrating motor controllers is similar, and discussed in more detail [later on in this document](#calibrating-the-controller).

Open [calibrate.js](https://github.com/tessel/servo-pca9685/blob/master/examples/calibrate.js)/copy and save the code below and run using `tessel run ./your/path/to/calibrate.js`. The code allows you to find the PWM duty cycle numbers that correspond to full forward, full reverse, and the neutral point by typing numbers into the command line and watching the result.

```js
var tessel = require('tessel');
var servolib = require('servo-pca9685');

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

Play around with the console to try out different values. Work your way slowly outwards to minimize the risk of damaging the servo by stalling out, drawing too much current, and frying something. Your servo is stalling if you can hear it trying to move but failing to. This happens when the servo is under heavy load ("working really hard"), and will happen when the servo runs into its built-in mechanical stops at the limits of its range of motion. When it happens, you should be able to hear the motor inside the servo struggling. 

For reference, I found that this particular servo had lower and upper limits of 0.0275 (2.75%) and 0.1225 (12.25%), which allowed it to move through its full range of motion:

![servo at one extreme](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/servo-module-tutorial/servo-low.jpg)

![servo at the other extreme](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/servo-module-tutorial/servo-high.jpg)

Once you've found values that work nicely for a particular physical servo or motor controller, save them!

##### **The command [`servo.configure( whichServo, minPWM, maxPWM, callback() )`](https://github.com/tessel/servo-pca9685#api-servo-configure-whichServo-minPWM-maxPWM-callback-Sets-the-PWM-max-and-min-for-the-specified-servo) maps the given max and min PWM values to 1.0 and 0.0, respectively, thereby making it easy to command two different devices to equivalent states.**

In the case of the values I found, the call would look like this:

```
servo.calibrate(1, 0.0275, 0.1225, function () {//callback goes here});
```

I could then call `servo.move(1, 0)` to set the servo to the lower extreme and `servo.move(1, 1)` to move it to the other extreme.

## Arbitrary actuators

Just because your motor doesn't have a "servo connector" doesn't mean it can't be used with Tessel. Everything in the image below is Tessel compatible!

![all the motors](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/servo-module-tutorial/motor_materials.jpg)

*Pictured: a servo power adapter (5V, 1A), a Tessel, a servo module, a small brushed DC gear motor (with clutch and plastic gear mounted), a brushed DC motor controller ([Sabertooth 2x60](http://www.dimensionengineering.com/products/sabertooth2x60)), a standard servo (the one that ships with the servo module), a micro servo, some 0.1" male to female jumper wires, a small scredriver (for hooking things up to the motor controller), a 6" micro USB cable, and a brushed DC wheelchair motor.*

### First things first

The first step to being able to control something is picking the actuator, and the second step is picking the controller. In this case, I have a small gear motor and a very large wheelchair motor. Both are brushed DC motors, so I'll be using the same controller, a [Sabertooth 2x60](http://www.dimensionengineering.com/products/sabertooth2x60) for both. To give you a sense of scale, the controller is overkill for the small motor but about right for the wheelchair motor in terms of its voltage, current, and power rating. I'll also be using the 12V bus of an ATX power supply to power the motor controller and, because old habits die hard, I've wired an E-Stop in and fused the wheelchair motor for good measure.

If your power source is a PC power supply like mine, you'll have to be careful about current surges, which happen when motors accelerate quickly. The supply I was using would shut itself off automatically if I stopped or started the wheelchair motor too suddenly, so I recommend starting small and calibrating the controller with a small motor if you have one on hand. I also recommend hooking everything up before you power anything on or run any code. It's also worth mentioing that you should **never** use the GPIO bank to power a motor or a motor controller. Appropriate use of the GPIO bank Vin pin can be found in [this document](./powering-tessel.md).

### Setup

![small motor setup](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/servo-module-tutorial/small_motor_setup.jpg)

The setup for the small motor, above, looks a lot like the setup for the large motor, below. In both cases, power comes in at the bottom left of the frame and runs through the E-Stop (big red button) before connecting the to motor controller. The motor wires are connected to channel 1 on the motor controller.

![big motor setup](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/servo-module-tutorial/big-motor-setup.jpg)

Because the motor controller takes standard servo PWM pulses, wiring the control signal from the servo module was super easy: I connected the ground/0V pin on the Tessel (labeled "-" on the servo module) to that of the motor controller and routed the servo signal lines (labeled "S" on the servo module) to the signal input pins on the motor controller. Here's a [short video clip](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/servo-module-tutorial/motor_move.mov) of the big motor running you can download if you're curious (not much to look at, honestly). 

### Calibrating the controller

I began by calibrating the motor controller using the small motor and [calibrate.js](https://github.com/tessel/servo-pca9685/blob/master/examples/calibrate.js). In the case of DC motors, the PWM duty cycle typically maps to velocity, rather than to position as it does in servos. A minimum duty cycle (`servo.move(servoNum, 0.0)`) commands full reverse, a maximum duty cycle (`servo.move(servoNum, 1.0)`) commands full speed ahead, and the PWM duty cycle in the middle (`servo.move(servoNum, 0.5)`) commands no speed at all.

The trick, then is finding the duty cycles that maximize the range of commandable speeds but don't saturate the inputs to the controller. Suppose that the motor controller in question expects duty cycles from 6% to 13%, with 6% meaning "full reverse" and 13% meaning "full forward". If you were lazy and called 5% and 15% good enough, you would not only have an asymmetrical speed response (`servo.move(servoNum, 0.75)` would be a different speed than `servo.move(servoNum, 0.25)`), the midpoint PWM value set when you called `servo.move(servoNum, 0.5)` of 10% would have the motor drift forward ever so slightly, as opposed to stop.

If all else fails, a good way to achieve balanced, albeit not necessarily maximum, coverage of avaialable speeds is to find the midpoint and then offset the min and max PWM values symmetrically from tham point. This approach might be desirable from a control perspective.

### Using your controller

It's best that the motors are stopped when the script starts and stops. Similarly, abrupt changes in velocity are not only physically jarring to the motor (and whatever it's connected to), but are also **much** more likely to damage electrical systems. Accelerate slowly and never go from full forward to full reverse all in one go.
