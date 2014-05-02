# Tessel hardware documentation

## What's here?

*  Physical overview of Tessel
*  Pin information for Tessel (board version TM-00-04)
*  Module usage, markings, and design philosophy
*  Key ICs (integrated circuits) on Tessel and links to datasheets
*  License information

## Physical overview

![Tessel with notable hardware features labeled](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/hardware_design_docs/TM-00-04-ports.png)

Tessel is 2.559" (65 mm) long and 2.185" (55.5 mm) wide. These numbers include the USB port and module headers, which protrude an extra 0.059" (1 mm) and 0.630" (16 mm) from the board edge. Tessel is 0.472" tall (12 mm).

There are four 0.13" (3.3 mm) diameter mounting holes in the corners of the board, each 0.1" (2.54 mm) in from the edges. They are 2.359" (59.92  mm) and 1.375" (34.93 mm) apart in the horizontal and vertical dimensions, respectively.

## Pins and ports

### Module ports

Tessel is extensible via modules, which connect to the 10-pin, 0.1” (2.54 mm) right angle female headers on the main board. 

* All pins use 3.3V logic.
* Module bank names (A, B, C, and D) are located between the module headers.
* Adjacent module ports are currently spaced apart by 0.3” (7.62 mm).
* Pins are located 0.089” (2.22 mm) from the board edge.

The following is the pinout for a module bank: 

Pin | Name | Notes
----|------|----
1 | GND  | Ground. Pins are marked with a circle.
2 | 3V3  |  3.3 V power rail. The onboard regulator is rated to 3A.
3 | SCL  | I2C clock line. Ports A and B and the GPIO bank share a common I2C bus, as do ports C and D.
4 | SDA  | I2C data line. Ports A and B and the GPIO bank share a common I2C bus, as do ports C and D.
5 | SCK  | SPI clock line. Common for the entire board.
6 | MISO  | SPI master in/slave out. Common for the entire board.
7 | MOSI  | SPI master out/slave in. Common for the entire board.
8 | GPIO1 / UART TX*  | User-configurable general purpose input/output. Unique to each module port. / UART serial transmit. See note below.
9 | GPIO2 / UART RX*  | User-configurable general purpose input/output. Unique to each module port. / UART serial receive. See note below.
10 | GPIO3  | User-configurable general purpose input/output. Unique to each module port.

*Note:* Module ports A, B, and D have hardware UART. Port C and the GPIO bank do not support UART communication at the time of this writing, but software implementation of UART is planned. Software UART will likely operate at 9600 BAUD (much slower than hardware UART) and consume significant computational resources, so we advise using it only if absolutely necessary.

### GPIO bank

* The bank itself is marked with "GPIO" on both ends.
* 2 x 10 (20 pins total).
* 0.1” (2.54 mm) pin spacing.
* All pins use 3.3 V logic.
* Although it is not recommended, the bottom row of the GPIO bank (nearest the board edge) can be used as a module port (port E on the Tessel).

Pin     |     Name  |  Notes
----|------|--------
1       |  GND     |   Ground                                                                                               
2     |     VIN  |   This is the input to the onboard 3.3 V regulator, and typically comes either from USB or an external battery. As such, its voltage can range from ~3.4 V to 15 V. Please read the [Powering Tessel](./powering-tessel.md) documentation before using this pin. It is not recommended as source of significant current and should **not** be used to power the board.
3       |  3V3     |   3.3 V power rail. The onboard regulator is rated to 3 A.                                              
4     |     A6   |   ADC6. 10-bit ADC, referenced to GND and 3.3 V. Cannot function as anything else.
5       |  SCL     |   I2C clock line. Common for the entire board.                                                         
6     |     A5   |   ADC5. 10-bit ADC, referenced to GND and 3.3 V. Cannot function as anything else.
7       |  SDA     |   I2C data line. Common for the entire board.                                                          
8     |     A4   |   ADC5. 10-bit ADC, referenced to GND and 3.3 V. Reconfigurable as digital GPIO.
9       |  SCK     |   SPI clock line. Common for the entire board.                                                         
10    |     A3   |   ADC4. 10-bit ADC, referenced to GND and 3.3 V.
11      |  MISO    |   SPI master in/slave out. Common for the entire board.                                                
12    |     A2   |   ADC3. 10-bit ADC, referenced to GND and 3.3 V. Reconfigurable as digital GPIO.
13      |  MOSI    |   SPI master out/slave in. Common for the entire board.                                                
14    |     A1   |   ADC2. 10-bit ADC, referenced to GND and 3.3 V. Reconfigurable as 10-bit DAC.
15      |  G1      |   GPIO1. User-configurable general purpose input/output. Planned as software UART TX.                  
16    |     G6   |   GPIO6. User-configurable general purpose input/output. 
17      |  G2      |   GPIO2. User-configurable general purpose input/output. Planned as software UART RX.                  
18    |     G5   |   GPIO5. User-configurable general purpose input/output.
19      |  G3      |   GPIO3. User-configurable general purpose input/output.                                               
20    |     G4   |   GPIO4. User-configurable general purpose input/output. 

### Power in pins ("VIN headers")

![vin headers](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/hardware_design_docs/vin.png)

Tessel includes a pair of pins for powering the board from a non-USB power source. These pins, referred to as the "VIN headers" are adjacent to the USB port. Please read the [Powering Tessel](./powering-tessel.md) documentation before using these pins.

### Advanced features

Tessel also includes a number of pins and pads which can be used to access the hardware at a lower level. These features are not intended for normal use and are not populated by default. They include:

*  Breakouts for the Reset and Config buttons (T7 and T1, respectively). Wiring in to these pins allows the user to control these inputs electronically.
*  JTAG headers (0.05" pitch, 2x5 grid), located between the LPC1830 and Module Ports A and B. JTAG is a common in-circuit debugging and programming tool.
*  A six-pin, 0.05" pitch debug header for the CC3000. Pinout (pin 1 is closest to Module Ports C and D, pin 6 is nearest the LPC):
  1.  CS
  2.  MISO
  3.  IRQ
  4.  MOSI
  5.  SCK
  6.  Enable
*  A footprint for an MMCX connector (small, robust, coaxial RF). Populating this connector and moving C18 to C28 allows the use of an external antenna. We recommend also buying an MMCX-to-SMA adaptor cable and 2.4 GHz SMA antenna.
*  Unexposed traces connected to a debug UART port on the CC3000. Please post in the forums or contact us if you think you need access to these pins, as accessing them is non-trivial.

## Modules

An overview of all first-party modules can be found [here](./modules-overview.md).

Module can be plugged into any module port and installed using Node Package Manager via the command line. Note that the BLE, GPRS, GPS, and Camera modules should not be used on Module Port C if possible due to their reliance on UART.

Although modules may occupy more than one physical port (such as the RFID and GPRS modules), they typically only use the electrical connections on one port. The other will be broken out to another header bank on the board.

### Module markings

![A Beta Tessel (TM-00-00) "fully loaded" with four modules: top ](./beta/images/TM-00-00-fullyloaded-top.jpg)

The module name is the top of board, usually in the upper left corner (with the pin headers facing left). This is also the name of the module's npm software package and is the name of the corresponding repository on GitHub.

![A Beta Tessel (TM-00-00) "fully loaded" with four modules: bottom](./beta/images/TM-00-00-fullyloaded-bottom.jpg)

The bottom silkscreen has, from top left to bottom right:

* The module's symbol, used to identify it at a glance
* The module name (same as on the top)
* The module model number
* The module's peak (as opposed to average) current draw
* The URL for the [Tessel website](https://tessel.io) 

*Note:* the Tessel's onboard 3.3 V regulator is rated to 3 A and that the typical base system requires between 150mA and 300mA depending on network usage and memory access. Putting parts of the Tessel into low power states to reduce power consumption is an area of active development.

Please note that the hexagon logo on each of the first-party modules is a trademark of Technical Machine. Although Technical Machine is still formulating third-party branding documentation and standards, at this time we request that our community reserves the hexagon for first-party and/or approved hardware.

### Module design philosophy

The module ports each include power, I2C, SPI, and three GPIO lines, which allows for the design of modules of arbitrary complexity. Note that the I2C and SPI busses are shared; as such, it is recommended that modules communicate over SPI and GPIO whenever possible. Doing so will mitigate the risks of I2C address conflicts and allow for multiple instances of the same kind of module.

Modules should be devices with clear-cut functionality. That is to say, they should have a single, well-defined purpose or a set of closely related functions, rather than an eclectic mix of capabilities onboard. This requirement is designed to reduce complexity, cost, and power consumption and maximize reusability in hardware and software.

If you're thinking of rolling your own module (or Tessel-compatible microcontroller), drop us a line at [team@technical.io](mailto:team@technical.io). We'd love to talk!

## Notable ICs

### LPC1830 - Tessel's processor

*  [Datasheet](http://www.nxp.com/documents/data_sheet/LPC1850_30_20_10.pdf)
*  [User manual](http://www.nxp.com/documents/user_manual/UM10430.pdf)
*  Cortex M3
*  Clocked at 180 MHz

### CC3000 - WiFi radio

*  [Datasheet](http://www.ti.com.cn/cn/lit/ds/symlink/cc3000.pdf)
*  [Product page](http://www.ti.com/product/cc3000)
*  [Wiki](http://processors.wiki.ti.com/index.php/CC3000)

### S25FL256SAGNFI001 - 32 MB flash memory

*  [Datasheet](http://www.spansion.com/Support/Datasheets/S25FL128S_256S_00.pdf)
*  Clocked at 90 MHz

### IS42S16160D-7BLI - 32 MB RAM

*  [Datasheet](http://www.issi.com/WW/pdf/42-45S83200D-16160D.pdf)
*  Clocked at 90 MHz

## License
Designed by [Technical Machine](http://technical.io/) and shared under a [Creative Commons Attribution ShareAlike license](http://creativecommons.org/licenses/by-sa/3.0/). All license text and links must be included in any redistribution.
