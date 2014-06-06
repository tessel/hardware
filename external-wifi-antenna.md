# How to add a big antenna to Tessel

## Overview

Tessel ships with a built-in chip antenna ([datasheet](http://www.johansontechnology.com/images/stories/ip/rf-antennas/JTI_Antenna-2450AT43A100_1-04.pdf)) which provides a modest peak gain of 2.0 [dBi](http://en.wikipedia.org/wiki/DBi#Antenna_measurements). If this is insufficeint for your needs (if the Tessel consistently has trouble connecting to WiFi networks or drops its connection frequently), you might consider performing the modification shown in the tutorial which follows. Note that the Tessel can only be configured to use either an external antenna or the built-in chip antenna, never both at the same time. This process is reversible, but requires delicate soldering.

Tests performed in-house, which used the [`RSSI` (recieve signal strength indicator)](http://en.wikipedia.org/wiki/Rssi) value returned by the CC3000, showed a greatly increased RSSI on Tessels using an external antenna compared to those using the built-in antenna. The grain of salt here is that the CC3000's RSSI reporting is not well documented and the values are unitless (and possibly [dimensionless](http://en.wikipedia.org/wiki/Dimensional_analysis) too).

For reference:
* The reported RSSI jumped from an average of ~51 to an average of ~69 with the antennas used in this tutorial. If the numbers are to be interpreted on a log scale (if they were somehow related to [dB](http://en.wikipedia.org/wiki/Decibel)), this difference would likely represent a considerable increase in signal power.
* We were able to see roughly twice as many networks with an external antenna as with the chip antenna (~9-11 vs. ~4,5), which makes sense given the observed signal strength boost at 2.4GHz from the external antenna. 

*Fair warning:* Whatever nonexistent warranty you may have thought Tessel came with would be voided if you took a soldering iron to your board, and this procedure is difficult even for people with considerable soldering experience. That said, the only thing you really risk damaging is the Tessel's ability to connect to WiFi.

### Materials

You'll need to following parts and tools in order to perform the modifiction:

#### An MMCX connector (through hole)

We used [this](http://www.digikey.com/product-detail/en/CONMMCX001/CONMMCX001-ND/1277204) MMCX female connector on the Tessels in the demo and it worked great.

#### An antenna and supporting hardware

There are a bunch of options here. You can buy antennas that connect directly to the MMCX connector (anything you find on Amazon for 'MMCX 2.4GHz antenna' should do) or you can find ones that use other connectors, such as SMA.

We opted for antennas with SMA connectors because the ones with MMCX connectors looked either flimsy or awkward to us, so we also needed this [MMCX to SMA adaptor cable](https://www.sparkfun.com/products/285).

*Note:* Most "normal" antennas for consumer 2.4GHz routers use RP-SMA connectors, as opposed to SMA or MMCX connectors. This means that the adaptor cable linked above will not work with the antenna from your old Linksys.

**Known-good SMA antennas:**

* [Full wave](http://www.digikey.com/product-detail/en/ANT-2.4-OC-LG-SMA/ANT-2.4-OC-LG-SMA-ND/2651620)
* [Half wave](http://www.digikey.com/product-search/en?x=15&y=13&lang=en&site=us&KeyWords=ANT-2.4-CW-QW-SMA-ND)
 
In testing, we found no significant difference between the full and half wave antennas in terms of reported RSSI.

#### Soldering iron, solder, and maybe flux and wick

Something high power (50 Watts or more) and with a fine tip. We love our [Weller WES51](http://www.digikey.com/product-detail/en/WES51/WES51-120V-ND/526397) and its accompanying [ETS tip](http://www.digikey.com/product-detail/en/ETS/ETS-ND/1669282), as well as [Kester solder paste](http://www.digikey.com/product-detail/en/7016070520/KE1507-ND/365533) (fine pitch spool solder works too). We have [ChipQuik](http://www.digikey.com/product-detail/en/SMD291/SMD291-ND/355201) flux and [Quick Braid](http://www.digikey.com/product-detail/en/Q-C-5AS/EB1090-ND/350545) on our lab bench.

#### Tweezers

Eric uses [these](http://www.amazon.com/gp/product/B006RBAHWM/ref=oh_details_o07_s02_i00?ie=UTF8&psc=1) fine tip tweezers for pretty much everthing. The tutorial shows images of [these](http://www.amazon.com/gp/product/B0015T787I/ref=oh_details_o07_s01_i00?ie=UTF8&psc=1) (which are sadly unavailable as of the time of this writing). 

## Adding the antenna

This modification connects an alternate RF path and populates the associated connector, thereby letting you add your own antenna.

#### Gather the materials and read this guide completely

![Step 1](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/wifi-01.jpg)
Pictured here are the MMCX connectors, the MMCX to SMA adaptor, the full wave antenna linked earlier in this tutorial, some tweezers, and a Tessel.

#### Get to know the neighborhood

The image below shows the area you'll be reworking: the RF output circuitry attached to the CC3000. Notable parts include:

* **U4** - the onboard chip antenna
* **C19, L4, C18** - the RF matching network we'll be modifying
* **J9** - the footprint for the MMCX connector
* **C28** - the footprint for an 0402 capacitor

![step 2 - the neighborhood](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/wifi-02.jpg)

#### Remove C18

Using a combination of the iron and tweezers, carefully remove C18 from the board (this is the hard part). When the deed is done, the Tessel should look like this:

![step 3 - remove c18](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/wifi-03.jpg)

Clean the pads for C18 and C29 as needed.

#### Populate C28

Take the capacitor you just pulled off and put it onto the previously empty pads labeled C28. In essence, you will have just rotated the capacitor by 90 degrees. If you lost C18 in the process, a [10pF ceramic 0402 rated to at least 10V](http://www.digikey.com/product-detail/en/CL05C100JB5NNNC/1276-1139-1-ND/3889225) should do.

A view through a microscope of what your Tessel should look like now:

![step 4 - through the microscope](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/wifi-04.jpg)

#### Populate J9, the MMCX connector

Put the MMCX connector into the J9 footprint...

![step 5 - top view](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/wifi-05.jpg)

...then flip the board over and solder the connector into place. Be careful that the connector is level and flush with the board when you solder it.

![step 5 - bottom view, soldered](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/wifi-06.jpg)

#### Hook it up!

The details here will vary depending on what kind of antenna you got, but the gist is "go ahead and plug it in".

In this case, I first plugged in the MMCX to SMA adaptor (it takes some effort to insert and remove; you may want pliers), then screwed on the antenna to the adaptor.

![step 6 - MMCX adaptor and populated connector](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/wifi-07.jpg)

![step 6 - MMCX to SMA adaptor in the board](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/wifi-08.jpg)

![step 6 - all done](https://s3.amazonaws.com/technicalmachine-assets/doc+pictures/wifi-09.jpg)

You're done! Go find some new networks!

## Switching back to the chip antenna

If for whatever reason you decide to move back to the onboard antenna, all you need to do is rotate C28 back onto the pads of C18. The presence of the MMCX connector doesn't matter (and good luck removing it).
