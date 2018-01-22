wiringPi BananaPI m2u & Berry ISR enabled
=========================================

This fork of the BPI-WiringPi2 enables ISR at least for the BPI m2u and Berry.
SoC: Allwinner R40/V40

Why another fork?
-----------------

In contrast to the RaspberryPIs the BananaPIs lacking of support by a grown community or even their own manufacturer, and a 
comparable community isn't present yet. I just wanted to sniff 433MHz devices and therefore certainly used the wiringPi library.
But to recieve such rf-data by a simple receiver the wiringPi library make use of an interrupt service routine, which was simply
not implemented by sinovoip in their fork of the wiringPi repository.

Which pins are capable of processing interrupts?
------------------------------------------------

Unfortunately I couldn't find a single resource that simply lists all pins, which are providing the 
"edge" control interface by the driver. At least I can say that the edge-interface is only 
exposed for gpios directly connected to the SoC. For example the PH0 (aka pin 29 on the gpio socket, aka
gpio pin 224, aka GPIO5).

How to determine the right gpio pin?
------------------------------------

The most annoying thing if you a getting such a raspi clone, finding the right pins.
At first you have to understand the terminology and to differentiate socket-pin, gpio pin, pin name and pin function. 
A pin on the socket of your BananaPi has 
- a number on this socket, for example '29'. Then you look into the available informations about the pinout and you get 
- at least one or more functions of this pin. In case of 29 it is GPIO5/CSI1-D0. If you're lucky you also find out it's 
- name, PH0. And if you know how, then you can calculate the
- gpio pin number.

The trickiest part is the gpio pin number, because it is depending on the pinout of the SoC. Such a SoC is connected by a huge 
amount of pins respectively balls (it's called ball grid array) to the pcb. To refer to a single pin/ball of the SoC these are 
organised in banks or groups. For example the pin named PH0 is pin number 0 from the 'H'-group. For the Allwinner R40/V40 each group
consists of up to 32 pins. The groups are named ascending alphabetically, so the 'H'-group is the 8th group. To determine the gpio
pin number you have to calculate it by following formular:

Let the name 'PH0', then 'H' is the 8th letter in alphabet and '0' index within the group:

`(IndexOfGroupLetterInAlphabet - 1) * 32 + IndexWithinGroup = GpioPinNumber`

`( 8 - 1 ) * 32 + 0 = 224`

Manually test the pin
---------------------

The mainline kernel driver from sinvoip provide the /sys/class/gpio access method. You can simply `ls -l /sys/class/gpio/` to list all gpios,
which are already exported. If you `ls -l /sys/class/gpio/gpioXXX` and can find a `edge` node, then you have found a candidate for ISR. To test
other pins on your BananaPI you have to find out its name, calclulate the gpio pin number and then manually export it.

For example BananaPI pin 29 -> PH0 -> gpio nr 224:

`echo 224 > /sys/class/gpio/export && ls -l /sys/class/gpio/gpio224/`

If the `echo` fails with `resource or device busy` then this pin is not available at all or is simply in use.

wiringPi README
===============

Please note that the official way to get wiringPi is via git from
git.drogon.net and not GitHub.

ie.

  git clone git://git.drogon.net/wiringPi

The version of wiringPi held on GitHub by "Gadgetoid" is used to build the
wiringPython, Ruby, Perl, etc. wrappers for these other languages. This
version may lag the official Drogon release.  Pull requests may not be
accepted to Github....

Please see

  http://wiringpi.com/

for the official documentation, etc. and the best way to submit bug reports, etc.
is by sending an email to projects@drogon.net

Thanks!

  -Gordon
