Pluto is a programmable digital watch that re-uses case and LCD 
panel of the Casio® F-91W.  This is the hardware repo, for the 
software side of things, see [pluto-fw](https://github.com/carrotIndustries/pluto-fw).
Looking for pictures? [There you go.](photos/)
You can discuss the project and ask questions on the [EEVblog forum](http://www.eevblog.com/forum/oshw/diy-watch-based-on-the-casio-f-91w/)

# Features
- Displays time in decimal/binary/hexadecimal base
- Multiple alarms
- Multiple countdown timers
- Uses RTTTL ringtones for alarm sound
- Stopwatch
- Compass
- Speedometer (Enter distance, watch measures time and displays speed in km/h)
- Generation of time-based one-time passwords according to RFC 6238 (WIP)
- Menu-driven interface
- Infrared receiver for software updates and TOTP secret transfer (WIP)
- Useless customisation (Key beep frequency, etc.)
- greater than 1 year battery life (estimate based on current consumption)


# Hardware
Pluto replaces the PCB of the Casio F-91W with a custom one containing 
a MSP430FR6972 MCU, a digital magnetometer (compass) and a 38 kHz 
infrared receiver for software updates and communication. Due to 
careful planning and luck, the Pluto PCB perfectly fits inside of the 
F-91W's mounting frame, although it houses many more components.
[Schematic as PDF](f-91w.pdf) [BOM](f-91w.bom)
[BOM with Mouser part numbers](bom_mouser.txt)

## MCU
The MSP430FR6972 is a low-power 16bit MCU with integrated segment LCD 
controller. It can be programmed and debugged using the two-wire 
SBW interface accessible through holes in the mounting frame. 

## Digital Magnetometer
The MAG3110 compass is connected to the MCU via I²C. To reduce its power 
consumption to zero when not in use, it's powered from GPIO pin. Due to 
layout constraints, the MCU's I²C interface can't be used, so the 
MCU's firmware has to implement a bit-banged I²C master controller.

## Infrared Receiver
The TSOP57338 38kHz IR receiver is powered from GPIO pin as well since 
it doesn't have any power saving modes. Transmitter and firmware 
support are WIP.

## Other components
The backlight LED is connected to two GPIO pins, so that LEDs of either 
orientation can be fitted.
The driving circuit for the buzzer uses an inductor (measured approx. 
10mH) to boost the battery voltage for increased volume.

## PCB
We manufactured the PCB at [elecrow](http://www.elecrow.com). PCB 
thickness has to be 0.6mm for the PCB to fit between battery clip and plastic 
frame. Note that the PCB has "castellated vias" (for contacting the 
button springs) as PCB manufacturers usually surcharge for these. We 
chose to use ENIG (gold) finish since the original PCB is gold-plated 
as well and Casio wouldn't have done so if this wasn't strictly 
necessary. Unfortunately, not all vias are covered with solder mask due 
manufacturing details (in the gerber files, they are), so you'll have 
to insulate the bottom side of the PCB from the battery with sticky 
tape.


# Software
For details on the software, see the software repository at 
[pluto-fw](https://github.com/carrotIndustries/pluto-fw).

# Why?
I used to own an ez430-Chronos watch, but after several years of use, 
the rubber wrist strap disintegrated, so I needed a new programmable watch. 
Pretty much all of the existing DIY watches are impractical for daily 
use because of bulk, being not waterproof or having miserable battery 
life. 

# PAQ (Possibly asked questions)
Q: Why not just use a smartwatch?  
A: Smartwatches have a miserable battery life of one week at best. Some 
even require user interaction to display the time.

Q: Why isn't there some sort of radio interface? (BLE, nRF24L01)  
A: Take look at [this](photos#perfect-fit) picture to get an idea of the space available 
inside of the F-91W frame. The nRF24L01 is 4 mm×4 mm and barely fits, but 
requires external components like balun and antenna. There's no space 
for these. An IR receiver is the same size as the nRF24L01, but doesn't 
require any external components. TI's CC430 MCUs seem like a perfect 
match for pluto as since they combine MCU and radio, but since the radio operates at 
sub-1GHz frequency, the antenna needs to be prohibitively large.

Q: Why is there no accelerometer?  
A: Accelerometers are the most useful when they're permanently active 
for applications like sleep trackers, pedometers or features like 
"shake to do something". Unfortunately, even accelerometers marketed as
"ultralow power" draw approx 30 µA when running, draining the CR2016
coin cell in less than half a year.  So we decided to dedicate the 
available space to a digital compass.

# Credits
Thanks go to [sh-ow](https://github.com/sh-ow) for adding the compass 
and helping to get the board ready for manufacturing.

# Mixed bag
Stuff loosely related to this project

## Reverse Engineering bare glass LCD panels
For basics on bare glass LCD panels and how to drive them,
see application notes from various MCU vendors.

For driving an unknown LCD panel, one needs to know which 
pins connect to the backplanes (COM) and which ones connect to segments 
(SEG). The easiest way to do so is to apply a square to all pins of the 
LCD panel and invert one pin at a time. When few segments light up, 
you've got a SEG pin, when many light up, you've got a COM pin. 
Sometimes apparently independent LCD segments are driven as one segment 
to save SEG pins. 


## Reverse engineering intricate PCB outlines
The F91-W's original PCB has a rather complicated outline that needs to 
be duplicated with high precision, so that the Pluto PCB fits nicely 
within the existing mounting frame.

By desoldering the spring contacting the piezo buzzer, the PCB is 
completely flat on the bottom side, so it can be placed on a flat-bed 
scanner. This eliminates any kind of distortion you'd get by taking a 
photo. The scanned image can then be scaled to match the actual 
dimensions measured with a caliper. There may be unequal scaling in X 
and Y direction due to inaccuracies of the scanner. We've now got a 
pretty accurate digital representation of the PCB that can be used for 
tracing out the outline and registering photos from the top side of 
the PCB to get the position of pads on top side.

Inkscape and GIMP (for correcting camera distortion) work well for 
these tasks. The resulting outline has been exported as a dxf document 
for importing the outline in KiCad.

## Things that didn't work out
The project started off with the intent of re-purposing the the F-91W 
case and LCD. Unfortunately, the LCD panel of the F-91W is rather 
limited since some LCD segments are driven as one segment.

So we went out and took a look at the Casio W800, F-201 and DB-36 
watch. After manufacturing a test jig for the LCD panels of the former 
two, it turned out that people at Casio /really/ like building LCDs 
with 5 backplanes. For some reason, the LCD panels need also to be driven 
with around 5 Vpp to achieve decent contrast. (Best guess: These 
watches are old designs, LCDs weren't that sensitive back then)

To my best knowledge, there's no (non-unobtainium) MCU on the market, 
that can boost a 3 V supply to 5 V for driving and LCDs /and/ can do 
5-mux. The only MCUs meeting the first requirement are from the RL78 
family from Renesas, but they only do 1,2,3,4 and 8-mux. Driving a 5-mux LCD 
with 8-mux waveforms leads to bad contrast. With 3 V and 5-mux, the contrast is 
much worse.

EPSON offers some obscure MCUs that can do both 5-mux and 
5 V LCD drive, but these ones don't have enough segment pins the QFN64 
package.

So we went back to F-91W.

## Measuring current consumption
For plotting current consumption over time, the EnergyTrace feature of 
the MSP430FR4133 LaunchPad has proven to be really useful: 
[energytrace-util](https://github.com/carrotIndustries/energytrace-util)

## Related Projects
Looking for more buttons and a radio? Take a look at [goodwatch](https://github.com/travisgoodspeed/goodwatch/) 
