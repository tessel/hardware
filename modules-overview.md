## Hardware documentation for Tessel modules

### Scope

This document provides hardware overviews for each Tessel module:

*  Accelerometer
*  Ambient
*  Audio
*  Bluetooth Low Energy
*  Camera
*  Climate
*  GPS
*  GPRS
*  Infrared
*  MicroSD Card
*  nRF24
*  Relay
*  RFID
*  Servo

**Design files and schematics coming soon.**

### The modules

#### Accelerometer

Measures acceleration in the x, y, and z directions.

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-01-XX
Latest version | TM-01-02
Key components | [MMA8452Q](http://www.freescale.com/files/sensors/doc/data_sheet/MMA8452Q.pdf)
Current consumption (rated max) | 165 microamps
Current consumption (average) | 165 microamps
Communication protocol | I2C, GPIO

##### Notes
*  I2C address can be modified by shorting J3
*  +x = towards the Tessel, +y = G3 --> GND, +z = up

#### Ambient

Measures ambient light and sound. Sound gain adjustable via R6. 

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-08-XX
Latest version | TM-08-03
Key components | [PT15-21C/TR8](http://www.everlight.com/datasheets/PT15-21C-TR8_datasheet.pdf) photodiode, [SPU0410HR5H-PB](http://ke-www2.knowles.com/search/prods_pdf/SPU0410HR5H.pdf) microphone
Current consumption (rated max) | 10 mA
Current consumption (average) | 10 mA
Communication protocol | SPI, GPIO

##### Notes

*  Light signal path includes low pass filters with f_c = 1.6 Hz
*  Sound signal path measures loudness via a half wave rectifier
  
#### Audio

Audio recording and playback

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-06-XX
Latest version | TM-06-05
Key components | VS1053B-L ([datasheet](http://www.vlsi.fi/fileadmin/datasheets/vlsi/vs1053.pdf), [product page](http://www.vlsi.fi/en/products/vs1053.html)) audio codec chip, [SPU0410HR5H-PB](http://ke-www2.knowles.com/search/prods_pdf/SPU0410HR5H.pdf) microphone
Current consumption (rated max) | 60 mA
Current consumption (average) | 40 mA
Communication protocol | SPI, GPIO

##### Notes

*  Supports both headphones (referenced to ~1.85 V) and line-out (referenced to GND) with a 32 Ohm max load
*  Audio recording through an onboard microphone or line-in jack (referenced to GND)
*  Both of the above are configured electronically via our API
*  Line-in can be used as a 16-bit ADC (feature not exposed in current APIs)

#### BLE

Communicate with BLE devices

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-07-XX
Latest version | TM-07-04
Key components | [BLE113-A](http://www.bluegiga.com/en-US/products/bluetooth-4.0-modules/ble113-bluetooth--smart-module/documentation/) radio
Current consumption (rated max) | 23 mA
Current consumption (average) | 18 mA
Communication protocol | UART, GPIO

##### Notes

*  A slave I2C port, an ADC, and two GPIOs are broken out from the BLE113 module's internal chip
*  The BLE113 chip can be programmed (firmware upload) using TI hardware and BlueGiga's toolchain

#### Camera

Take pictures with your Tessel

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-12-XX
Latest version | TM-12-04
Key components | LCF-23TN ([datasheet](https://dl.dropboxusercontent.com/s/vm5lnjo0mhta7u1/LCF-23TN%20DATASHEET%28SPI%20JPEG%20Module%29.pdf), [manual](https://www.dropbox.com/s/q1og0ug68a4f9t8/LCF-23TN%20Protocol.pdf)) camera module, a clone of the [VC0706](http://www.southernstars.com/skycube/files/VC0706.pdf) video processor IC
Current consumption (rated max) | 90 mA
Current consumption (average) | 90 mA
Communication protocol | SPI, UART, GPIO

##### Notes

*  Board also includes an ISP header pinout so that it could be used with cameras like the [Pixy](https://www.kickstarter.com/projects/254449872/pixy-cmucam5-a-fast-easy-to-use-vision-sensor). Populating the header would require removing the existing camera
*  Much of the documentation is in Chinese. We apologize.

#### Climate

Measure temperature and humidity

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-02-XX
Latest version | TM-02-02
Key components | [SI7005-B-GM](http://www.silabs.com/Support%20Documents/TechnicalDocs/Si7005.pdf) sensor
Current consumption (rated max) | 565 microamps
Current consumption (average) | 320 microamps
Communication protocol | I2C, GPIO


#### GPS

Global positioning

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-09-XX
Latest version | TM-09-02
Key components | A2235-H ([user manual](http://www.mouser.com/catalog/specsheets/EVA2235-H.pdf), [product page](http://www.maestro-wireless.com/a2235-h)) GPS
Current consumption (rated max) | 69 mA
Current consumption (average) | 31 mA
Communication protocol | UART, GPIO

##### Notes

*  **Errata:** module is marked with average current draw of 31 mA on back silk. While not technically incorrect, this is inconsistent with other modules, which are marked with their peak current draw.


#### GPRS

Voice calls and SMS from Tessel

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-14-XX
Latest version | TM-14-01
Key components | [SIM900](http://wm.sim.com/producten.aspx?id=1019) GPRS radio
Current consumption (rated max) | 2 A
Current consumption (average) | 150 mA
Communication protocol | UART, GPIO

##### Notes

*  This is a "double-wide" module and it *can* use both ports if its jumpers in the top left corner are connected. When connected, G3 is an interrupt which fires when a call or SMS is received and G1 and G2 are a debug UART port (115200 baud only) used to update the SIM900's firmware. SPI and I2C are unused. More documentation will be made available for this soon.
*  Selectable power source via jumper in top right corner: 3.3 V from Tessel or external power regulated to (3.3 + 5% trim) V. The latter is recommended if possible.
*  The onboard (3.3 + 5% trim) V regulator is rated to 3 A and an absolute max of 17 V in.
*  Current consumption peaks during transmission and is dependent on output signal power
*  Replacement antennas will be available in the store
*  Bring your own SIM card. (We used one from an AT&T GoPhone)
*  Separate mic and headphone jacks (3.5 mm)

#### Infrared

Communicate and command household electronics 

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-11-XX
Latest version | TM-11-03
Key components | [TSOP38238](http://www.vishay.com/docs/82491/tsop382.pdf) 38 kHz IR demodulator, [SFH-4646-Z](http://www.osram-os.com/Graphics/XPic6/00116152_0.pdf) IR LED
Current consumption (rated max) | 95 mA
Current consumption (average) | 5 mA
Communication protocol | UART, GPIO

##### Notes

*  Receiving IR takes very little power. Transmission, on the other hand, uses a good deal of power.
*  Receiver frequency is fixed at 38 kHz, although pin-compatible parts which operate at other frequencies presumably exist. Check your appliance's manual and/or the internet to make sure your hardware is compatible.


#### Micro SD Card

Store your data locally (not in the magical, mystical Cloud)

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-05-XX
Latest version | TM-05-05
Key components | N/A
Current consumption (rated max) | 50 mA
Current consumption (average) | 5 mA
Communication protocol | SPI, GPIO

##### Notes

*  None

#### nRF24

Low power wireless communication

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-13-XX
Latest version | TM-13-01
Key components | [nRF24L01+ product page](http://www.nordicsemi.com/eng/Products/2.4GHz-RF/nRF24L01P)
Current consumption (rated max) | 12 mA
Current consumption (average) | 12 mA
Communication protocol | SPI, GPIO

##### Notes

*  Good for mesh networking

#### Relay

Switch large, high voltage loads

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-04-XX
Latest version | TM-04-04
Key components | IM02DGR ([DigiKey page](http://www.digikey.com/product-detail/en/IM02DGR/PB1171CT-ND/1828461))
Current consumption (rated max) | 90 mA
Current consumption (average) | 10 mA
Communication protocol | GPIO

##### Notes

*  Relays are normally open (load disconnected) and will open should the module/Tessel lose power
*  Current draw increases when the relays close (I-draw = 10mA + 40mA * [number of relays on])
*  Rated to 240 V, 5 A (but please be careful)

#### RFID

Read (and someday write) from (/to) 13.56 MHz RFID cards and tags

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-10-XX
Latest version | TM-10-03
Key components | PN532 ([datasheet](http://www.adafruit.com/datasheets/pn532longds.pdf), [user manual](http://www.nxp.com/documents/user_manual/141520.pdf), [MiFARE tag IC/protocol docs](http://www.nxp.com/documents/data_sheet/MF1S503x.pdf)) 13.56 MHz RFID transceiver
Current consumption (rated max) | 150 mA
Current consumption (average) | 60 mA
Communication protocol | I2C, GPIO

##### Notes

*  Module can be switched to communicate over SPI with appropriate resistor pop option swaps
*  Current is highest when transmitting and much lower most of the time
*  Compatible MiFare cards will be available in the store


#### Servo

Control up to 16 servos

##### Quick overview:

Parameter | Value
----------|------
TM part # | TM-03-XX
Latest version | TM-03-02
Key components | [PCA9685](http://www.adafruit.com/datasheets/PCA9685.pdf) LED driver
Current consumption (rated max) | 50 mA
Current consumption (average) | 10 mA
Communication protocol | I2C, GPIO

##### Notes

*  An I2C LED PWM driver makes for a great servo driver too!
*  16 channels
*  Tessel is a 3.3V device, so V_out on the PWM channels is 3.3 V (works for standard hobby servos)
*  5.5 mm barrel jack (center positive) for powering servos
*  Can be used to control anything that needs PWM (speed controllers, gate drivers-to-FETs-big LEDs, etc.)
